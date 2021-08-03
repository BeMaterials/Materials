# Authentication and Authorization

### Add User Entity

- In the `Domain` project, add `AppUser.cs`, install `Microsoft.AspNetCore.Identity.EntityFrameworkCore`, and restore.
- Add the props that do not already exist in the `IdentityUser` class.
- In the `DataContext` class, change `DbContext` to `IdentityDbContext<AppUser>` and add `base.OnModelCreating(builder);` in the `OnModelCreating` method.
- Add a new migration by `dotnet ef migrations add "AddedIdentity" -p Persistence -s API/`.

### Configure Identity in the API app

To enable API to create, manage, and login users via a UserManager and a signInManager services:

- In the `API` project, install `Microsoft.AspNetCore.Identity.UI`, and restore.
- in the `Startup` class, add:

```c#
services.AddDefaultIdentity<AppUser>().AddEntityFrameworkStores<DataContext>();
```

### Seed users

- In the `Seed.cs`, pass `UserManager<AppUser> userManager` to the `SeedData` method.
- The `userManager` is like a context for the users.
- Add logic to add some users.
- Create and pass a `userManager` to the `SeedData` method in the `Main` method.
- Restart the app.

### Login handler

- In the `Application` project, create `Users/Login.cs`.

```c#
public class Query : IRequest<AppUser>
{
    public string Email { get; set; }
    public string Password { get; set; }
}

public class QueryValidator : AbstractValidator<Query>
{
    public QueryValidator()
    {
        RuleFor(u => u.Email).NotEmpty();
        RuleFor(u => u.Password).NotEmpty();
    }
}

public class Handler : IRequestHandler<Query, AppUser>
{
    private readonly UserManager<AppUser> _userManager;
    private readonly SignInManager<AppUser> _signInManager;

    public Handler(UserManager<AppUser> userManager, SignInManager<AppUser> signInManager)
    {
        _userManager = userManager;
        _signInManager = signInManager;
    }

    public async Task<AppUser> Handle(Query request, CancellationToken cancellationToken)
    {
        var user = await _userManager.FindByEmailAsync(request.Email);

        if (user == null)
            throw new RestException(HttpStatusCode.Unauthorized);

        var result = await _signInManager
            .CheckPasswordSignInAsync(user, request.Password, false);

        if (result.Succeeded)
        {
            // TODO: generate token
            return user;
        }

        throw new RestException(HttpStatusCode.Unauthorized);
    }
}
```

- Create `UserController` (First introduce `BaseController` and refactor other controllers to use it).

```c#
[HttpPost("login")]
public async Task<ActionResult<AppUser>> Login(Login.Query query)
{
    return await Mediator.Send(query);
}
```

- Because it sends back all the info about the user and we don't want that, create a `UserDto.cs` in the `Application` project to serve as some kind of DTO and change the `UserController` and `Login` classes as well:

```c#
public class UserDto
{
    public string DisplayName { get; set; }
    public string Token { get; set; }
    public string Username { get; set; }
    public string Image { get; set; }
}
```

### JWT

We are going to create an infrastructure project (a classlib project) to generate a jwt for us:
![](/md/jwt.jpg)

- In the terminal:

```dos
dotnet new classlib -n Infrastructure
dotnet sln add Infrastructure/
cd Infrastructure
dotnet add reference ../Application
cd ../API
dotnet add reference ../Infrastructure
```

- Change the `TargetFramework` to `netcoreapp3.1` in the `csproj` file.
- Create `IJwtGenerator` interface inside `Application` project.
- Create the implementation of the above interface (`JwtGenerator`) in the `Infrastructure` project (For that, install `System.IdentityModel.Tokens.Jwt` in the `Infrastructure` project).
- Add `services.AddScoped<IJwtGenerator, JwtGenerator>();` in the `Startup.cs`.
- Modify `Login.cs` to return a jwt upon success.

### Securing API

- Add `Microsoft.AspNetCore.Authentication.JwtBearer` package to the `API` project.
- Add the following in the `Startup.cs`:

```c#
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer(opt =>
{
    opt.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("super secret key")),
        ValidateAudience = false,
        ValidateIssuer = false
    };
});
```

- Add `UseAuthentication` middleware in the `Configure` method in this order:

```
UseMiddleware<ErrorHandlingMiddleware> -> UseRouting -> UseCors -> UseAuthentication -> UseAuthorization -> UseEndpoints
```

- Use `[Authorize]` attribute on any method (route) that you want to protect.
- Now the consumer has to provide `Authorization` header with `Bearer ...` for each request to access it.

### User Secrets

- Run `dotnet user-secrets init -p API/` to set `<UserSecretId>` to a Guid.
- The command above will create a folder with the name of the newly generated guid in the local computer.
- Run `dotnet user-secrets set "TokenKey" "super secret key" -p API/`.
- To list all secrets: `dotnet user-secrets list -p API/`.
- Instead of the string, use `Configuration["TokenKey"]`.
- Because we don't have access to `Configuration` in the `JwtGenerator`, we inject `IConfiguration` in its constructor.

### Authorization policy

Add this option to the `AddControllers` method in the `Startup`:

```c#
opt =>
{
    var policy = new AuthorizationPolicyBuilder().RequireAuthenticatedUser().Build();
    opt.Filters.Add(new AuthorizeFilter(policy));
}
```

Now, all the routes requires authorization. So add `[AllowAnonymous]` to the `UserController` class and remove `[Authorize]` from previous methods.

### Register user

- Create `Register.cs` operation class in the `Application` project. This is the only command that returns something (a User object in this case).
- Create `Register` method in the `UserController`.

### Adding Fluent Validator extension for password

Create `ValidatorExtensions` class in `Application>Validators` to extend `IRuleBuilder`.

### Retrieve Username from a jwt

- Create `IUserAccessor` interface inside `Application` project.
- Create the implementation of the above interface (`UserAccessor`) in the `Infrastructure` project.
- Add `services.AddScoped<IUserAccessor, UserAccessor>();` in the `Startup.cs`.

Now you can implement a `CurrentUser` operation class and an action method inside `UserController`.
