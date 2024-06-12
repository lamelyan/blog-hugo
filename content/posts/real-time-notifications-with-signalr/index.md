+++
title = 'Real-time notifications with SignalR'
date = 2024-06-12T02:24:43-05:00
draft = false 
featured_image = 'signalr-dotnet.jpg'
toc = true
tags = ["csharp", "dot-net", "angular", "typescript"]
+++

Using real-time communication with SignalR allows users to interact with the application without needing to refresh the screen to get updates.

This post provides an overview of setting up SignalR in a .NET Core application. In this example, we’ll use a .NET Core API as the backend and Angular as the frontend.

We implemented this type of communication on dashboard pages to display workflow status details.

## Backend implementation 

### .NET Core API

After you include [SignalR package](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client#readme-body-tab) 
in your project, register it during startup.



```csharp
services.AddSignalR(config => config.EnableDetailedErrors = true)
```

Configure a 'hub' endpoint that is used to send out notifications

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
This is the name of a javascript function that will be invoked on the client. In our Angular app, 
we have a function with the name - UserNotification - that handles the notification object passed to it.


Also, note, that notifications are filtered to go to clients (browsers or other apps) that have specific id. 

### How does SignalR know about who is connected? 

When the frontend app connects to the backend, it passes an identifier using a JWT token. 
To retrieve this identifier, we implement the IUserIdProvider interface. 
This interface allows us to define what identifies a user. 
In our case, it's the ID stored in a claim of the JWT token.

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

Once implemented, SignalrUserProvider is registered at start up.

```csharp
services.AddSingleton<IUserIdProvider, SignalrUserIdProvider>();
```


By default, Azure AD includes user's id in the claim - NameIdentifier. 


### Browser limitation workaround

Due to browser API limitations, when establishing a connection via a WebSocket, 
the JWT token needs to be sent in a query string. Therefore, we need to add special logic in Startup.cs 
to read this token and add it to our context. This allows us to access token 
information (such as claims) to obtain details about the user.

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

Microsoft offers [@azure/msal-angular](https://github.com/AzureAD/microsoft-authentication-library-for-js#readme) 
package to assist us with Azure Active Directory B2C sign-in.

Using the msal-angular library, we can connect to the backend hub:



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

On the client side, we listen to events coming from the backend.
The event will pass an object with data that we can use to update the UI. 

To achieve this, create a helper method that starts a connection and listens for a specific event.


```typescript
  listenForEvent<T>(eventName, token): Observable<T> {
    const stream$ = new Subject<T>();
    this.initializeConnection(token);
    this.hubConnection.on(eventName, (message: any) => stream$.next(message));
    return stream$.asObservable();
}
```

Then you can listen for specific event and process event data. 

```typescript
private listenForUserNotifications() {
    this.reportNotification$ = this.notificationProvider
        .listenForEvent('UserNotification', this.auth.token).pipe(
        takeUntil(this.destroy$),
        finalize(() => console.log(`%cUnsubscribed from UserNotification`, 'color:yellow; background-color:seagreen')));
}
```

### Conclusion
In summary, by using SignalR libraries on both the client and server sides, 
we establish a WebSocket connection. It's essential to ensure that the authentication 
token is passed if the connection requires authentication. One particular gotcha is passing 
the token in the query string, which can be a security concern since some loggers will log queries.

Additionally, if SignalR cannot open a WebSocket, it falls back to polling. 
This convenient feature can also cause bugs to slip through the cracks since 
it’s not always obvious that things aren't working as expected. 
Therefore, make sure to have your web developer console open and check for errors as you test. 😉


