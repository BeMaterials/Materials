# Validation in the API and the Client

### Debugging

Make sure that the following is in the `launch.json` in the `.vscode` folder:

```json
{
  "name": ".NET Core Attach",
  "type": "coreclr",
  "request": "attach",
  "processId": "${command:pickProcess}"
}
```

If you want to debug something for the running process use `.NET Core Attach` and then select `API.exe` (if you want to debug something regarding startup use `.NET Core Launch`).

- `F9` for adding a `Breakpoint`
- `F10` to step over
- `F11` to step in to a method
- `shift + F11` to step out.

In the solution folder:

```dos
dotnet build
```

will build all the projects in the solution or you can use `dotnet build [ProjectName]` to build a specific project.

### Data annotation

The simplest form of validation (which we won't be using) is to use annotations on the properties in the `Command` class in each operation class in the `Application` project. In this way, because we used `[ApiController]` attribute, after the binding, because the `ModelState` prop of the current controller is not valid, it automatically responds with a bad request response.

### Fluent validation package

Add the `FluentValidation.AspNetCore 8.5.1` package to the Application project and restore.

![](/md/fluent_validation.jpg)

The common pattern is to write validation between the `Command` and `Handler` classes:

```c#
public class CommandValidator : AbstractValidator<Command>
{
    public CommandValidator()
    {
        RuleFor(c=>c.Title).NotEmpty();
    }
}
```

In the `Startup` class, change `services.AddControllers()` to this:

```c#
services.AddControllers().AddFluentValidation(config =>
{
    config.RegisterValidatorsFromAssemblyContaining<Create>();
});
```

### Error handling

In the current architecture, if there is an exception in the Application logic (for example a resource not found), because we don't have access to HttpContext instance (with the request and response props) we throw an exception which cannot be differentiated with a server error (500).

- Each middleware has a prop of type `RequestDelegate` which often is called `_next`.
- Each middleware has a method `Invoke` which will be invoked with an instance of `HttpContext`.
- `Configure` method in the `Startup` is used to configure the HTTP request pipeline by registering middlewares.
- Our EndPoints are the last middleware:

![](/md/middleware.jpg)

So a general error-handling middleware should be the first middleware:

- Create `ErrorHandlingMiddleware.cs` in the API/Middleware.
- Register it in `Configure` method.
