# SignalR

### API

It mainly uses WebSockets. The idea is to create a hub (instead of API endpoint) and clients make connection to it. Every time a message comes to hub, the hub notifies every subscribers.

- Create `Comment` entity, nav props, DbSet, and migration
- Create `CommentDto` and its mapper
- Create `Create` operation and its command and handler

  **Important**: because this has not a HttpContext (it is WebSocket), we have to add the `Username` in the command props instead of using `UserAccessor`.

- Register `SignalR` in the `Startup`: `services.AddSignalR();`
- Create `ChatHub` in the `API/Hubs` with a `SendComment` method (here, unlike the methods in the API controllers, the name of the method is important and is sent from the client side to invoke this particular method):

```c#
public async Task SendComment(Create.Command command)
{
    var username = Context.User?.Claims?.FirstOrDefault(c => c.Type == ClaimTypes.NameIdentifier)?.Value;

    command.Username = username;

    var comment = await _mediator.Send(command);

    await Clients.All.SendAsync("ReceiveComment", comment);
}
```

- In order to add the user token to the hub context we have to add the following option in the `AddJwtBearer` method:

```c#
opt.Events = new JwtBearerEvents
{
    OnMessageReceived = context =>
    {
        var accessToken = context.Request.Query["access_token"];
        var path = context.HttpContext.Request.Path;
        if (!string.IsNullOrEmpty(accessToken) && (path.StartsWithSegments("/chat")))
        {
            context.Token = accessToken;
        }

        return Task.CompletedTask;
    }
};
```

- Add `.AllowCredentials();` to the CORS policy.

- Register `ChatHub` in the router (`useEndpoints` in the `Startup`): `endpoints.MapHub<ChatHub>("/chat");`

### Client

```dos
npm i @microsoft/signalr
```
