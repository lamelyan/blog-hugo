+++
title = 'Real-time notifications with SignalR'
date = 2024-05-28T09:24:43-05:00
draft = false 
featured_image = 'signalr-dotnet.jpg'
toc = true
tags = ["csharp", "dot-net", "angular", "typescript"]
+++

Using real-time communication with SignalR allows users to interact with the application 
without needing to refresh the screen to get updates.

This post provides a detailed overview of setting up SignalR in a .NET Core application.
In this example, we'll use a .NET Core API as the backend and Angular as the frontend.

We implemented this type of communication on dashboard pages that display workflow status details.


## Backend implementation 

### .NET Core API

SignalR is injected at Startup  

```csharp
services.AddSignalR(config => config.EnableDetailedErrors = true)
```

A special 'hub' endpoint is configured to send out notifications

```csharp
app.UseEndpoints(endpoints =>
{
   endpoints.MapHub<NotificationsHub>("/notifications");
   endpoints.MapControllers();
});
```

Note the part `MapHub<NotificationsHub>` This isn't a regular REST API endpoint. 
This endpoint allows us to create a persistent bi-directional connection(using a WebSocket) between the server and clients.

### Notification Hub

What is a Hub? Hub is a part of SignalR API that enables the communication between the server
and clients in real-time. To use it, you extend the abstract class Hub and definite your logic.
In our case, we need to make sure that the Hub endpoint is authorized.
This means that the client must pass a JWT token to connect to the notification hub.

```csharp
[Authorize]
public class NotificationsHub : Hub
{
}
```

### Send notification

A hub can be injected as `IHubContext<NotificationsHub> hubContext` 
and used wherever we need to send out a notification.

Here is example of code that sends out notification to all users with specific Azure Active Directory Id:

```csharp
await _hubContext.Clients.Users(AzureActiveDirectoryId).SendAsync("UserNotification", new
{
//removed for brevity
}, cancellationToken);
```


Note the first parameter to the SendAsync method - "UserNotification".
This is the name of a javascript function that will be invoked on the client.  
In our Angular app, we have a function with the name - UserNotification - that handles 
the notification object passed to it.


Also, note, that notifications are filtered to go to clients (browsers or other apps) that have specific id. 

### How does SignalR know about who is connected? 

When the frontend app connects to the backend, it passes an identifier (using JWT token). 
To get this identifier, we implement IUserIdProvider interface. 
In it, we define what identifies a user. In our case, it's the id that is stored in a claim of JWT token.

```csharp
public class SignalrUserIdProvider : IUserIdProvider
{
    readonly IGetCurrentUserInfo _currentUserInfo;

    public SignalrUserIdProvider(IGetCurrentUserInfo currentUserInfo)
    {
        _currentUserInfo = currentUserInfo;
    }
    public string GetUserId(HubConnectionContext connection)
    {
        var authProviderId = connection.User.Claims
            .FirstOrDefault(x => x.Type == ClaimTypes.NameIdentifier)?.Value;
    
        return authProviderId;
    }
}
```

Once implemented, this provider SignalrUserProvider is wired up at a startup.

```csharp
services.AddSingleton<IUserIdProvider, SignalrUserIdProvider>();
```

By default, Azure AD will include user's id in the claim of type NameIdentifier. 


### Authentication 

To deliver real-time notifications, SignalR establishes a connection.

Because of browser API limitation, when establishing connection via websocket,
the JWT token needs to be sent in a query string.


And because of that,  there is specific logic in Startup.cs to read this token and add it to our context.

This allows us to read token information (such as claims) in order to get information about the user.

```csharp
static void ConfigureAuthenticationForSignalR(IServiceCollection services)
{
    services.Configure<JwtBearerOptions>(JwtBearerDefaults.AuthenticationScheme, options =>
    {
        var existingOnMessageReceivedHandler = options.Events.OnMessageReceived;
        options.Events.OnMessageReceived = async context =>
        {
            await existingOnMessageReceivedHandler(context);

            var accessToken = context.Request.Query["access_token"];

            // If the request is for our hub...
            var path = context.HttpContext.Request.Path;
            if (!string.IsNullOrEmpty(accessToken) &&
                (path.StartsWithSegments("/notifications")))
            {
                // Read the token out of the query string
                context.Token = accessToken;
            }
        };
    }); 
} 
```


## Frontend integration

In this example we use Angular as frontend framework. 
If you are using another frontend framework, Microsoft 
has helper libraries for all different frontend frameworks.


### Start SignalR connection

Microsoft has [@azure/msal-angular](https://github.com/AzureAD/microsoft-authentication-library-for-js#readme) 
package to assist us with Azure Active Directory B2C sign-in.

We connect to the backend hub, like so:



```typescript
 private initializeConnection(token: string) {
    this.hubConnection = new signalR.HubConnectionBuilder()
        .withUrl(this.config.baseUrl + '/../notifications',
            {accessTokenFactory: () => token})
        .withAutomaticReconnect([2000, 5000, 10000].concat(new Array(50).fill(30000)))
        .build();

    this.hubConnection
        .start()
        .then(() => console.log('SignalR - Connection STARTED.'))
        .catch(err => console.log('Error while starting SignalR connection: ' + err));


    this.hubConnection.onclose(_ => console.log('SignalR - Disconnected.'));
}
```

For more details see [Microsoft docs here](https://learn.microsoft.com/en-us/aspnet/core/signalr/authn-and-authz?view=aspnetcore-8.0#bearer-token-authentication).



### Handle notification events

Create a helper method that starts a connection and listens to a specific event.

```typescript
  listenForEvent<T>(eventName, token): Observable<T> {
    const stream$ = new Subject<T>();
    this.initializeConnection(token);
    this.hubConnection.on(eventName, (message: any) => stream$.next(message));
    return stream$.asObservable();
}
```

Here is a sample method that listens for UserNotification event:

```typescript
private listenForUserNotifications() {
    this.reportNotification$ = this.notificationProvider
        .listenForEvent('UserNotification', this.auth.token).pipe(
        takeUntil(this.destroy$),
        finalize(() => console.log(`%cUnsubscribed from UserNotification`, 'color:yellow; background-color:seagreen')));
}
```
