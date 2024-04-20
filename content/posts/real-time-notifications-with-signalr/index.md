+++
title = 'Real-time notifications with SignalR'
date = 2024-04-18T09:24:43-05:00
draft = true 
featured_image = 'feature-image.webp'
toc = true
+++

This is a detailed overview of how SignalR can be set up in .NET Core application. 

## Startup

SignalR is injected at Startup  

services.AddSignalR(config => config.EnableDetailedErrors = true);

A special 'hub' endpoint is configured to send out notifications

```csharp
app.UseEndpoints(endpoints =>
{
   endpoints.MapHub<NotificationsHub>("/notifications");
   endpoints.MapControllers();
});
```

Note the part `MapHub<NotificationsHub>` This isn't a regular REST API endpoint. 
This endpoint allows us to create a persistent bi-directional connection(using a web socket) between the server and clients.

## Notification Hub

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

## Send notification

A hub can be injected as `IHubContext<NotificationsHub> hubContext` 
and used wherever we need to send out a notification.

Here is example of code that sends out notification to all users with specific AzureActiveDirectoryId:

```csharp
await _hubContext.Clients.Users(AzureActiveDirectoryId).SendAsync("UserNotification", new
{
//removed for brevity
}, cancellationToken);
```


Note the first parameter to the SendAsync method - "UserNotification".
This the name of a javascript function that will be invoked on the client.  
In our Angular app, we have a function called UserNotification that handles 
the notification object passed to it.


## How does SignalR identify users?

SignalR hub enables us to control who should receive notification. 
In sample below, we tell it to send a noticiation to all the clients where user
is identified by a provided azure active directory id. 


```csharp
await _hubContext.Clients.Users(AzureActiveDirectoryId).SendAsync("UserNotification", new
{
//removed for brevity
}, cancellationToken);
```


Notifications are filtered to go to clients (browsers or other apps) that have specific id. 

### How does SignalR know which clients have this id?

To get the ID for a client that is currently connected, we implement IUserIdProvider interface. 
In it, we define what identifies a user. In our case, it's the id that is stored in a claim of JWT token

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

This SignalrUserProvider is wired up at a startup.

`services.AddSingleton<IUserIdProvider, SignalrUserIdProvider>();`

By default, Azure AD will include user's id in the claim of type NameIdentifier. 


## Authentication 

To deliver real-time notifications, SignalR establishes a web socket connection.

It's an authenticated connection and the authentication JWT token is sent as a query in URL.
In Startup.cs, there is specific logic to read this token and add it to our context.
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

