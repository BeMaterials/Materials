# Setup

```dos
dotnet new sln

dotnet new classlib -n Core
dotnet new classlib -n Persistence
dotnet new webapi -n Application

dotnet sln add Core/
dotnet sln add Persistence/
dotnet sln add Application/

dotnet sln list

cd Application
dotnet add reference ../Core/
dotnet add reference ../Persistence/

cd ../Persistence
dotnet add reference ../Core/

cd ../Application
rm WeatherForecast.cs
rm Controllers/WeatherForecastController.cs
```

- Change `netstandard2.0` to `netcoreapp3.1` in `Core.csproj` and `Persistence.csproj`.
- Comment out the `app.UseHttpsRedirection();` middleware in `Startup.cs`.
- Remove `https://localhost:5001;` from `Application/Properties/launchSettings.json` in the Application:applicationUrl section.

- To run the app from the root folder:

```dos
dotnet run -p Application/
```

- To run the app from the Application folder:

```dos
dotnet run
```

- To run the app from the Application folder in the watch mode:

```dos
dotnet watch run
```

- `csproj` files are the ones recognized by `Visual Studio` to see the `References` of each project.
- `Program.cs` has a `Main` method which creates a `Http server`.
- `ConfigureServices` method in the `Startup` class is used for `Dependency Injection`. So if the controllers have a dependency which we should initialize it in its constructor, we register it here. For example,

```c#
services.AddScoped<IRepository, Repository>();
```

This way we registered `Repository` as a concrete implementation of `IRepository` and in our code wherever we needed an instance of IRepository in the ctor, runtime will provide a new instance of Repository and inject it to that class. This is not limited to interfaces and their concrete implementations, we can pre-configure an instance and inject it to the controller class as well, like `DbContext`.

- `Configure` method in the `Startup` class is where we introduce `middlewares` in order to work with request.

# Adding Database

- We create the domain models in the `Core` project based on the relationships below and create the composite key in the `DataContext`.

  ![](/md/er.jpg)

- In `Persistence` install the following packages by using NuGet Package Manager:
  - "Microsoft.EntityFrameworkCore"
  - "Microsoft.EntityFrameworkCore.SqlServer"
- Then,

```dos
cd ..
dotnet restore
```

- To enable the dotnet-ef command line tool:

```dos
dotnet tool install --global dotnet-ef
```

- Create `DataContext` (or `ApplicationDataContext`) in the root of `Persistence` project:

```c#
public class DataContext : DbContext
{
    public DbSet<Make> Makes { get; set; }
    public DbSet<Model> Models { get; set; }

    public DataContext(DbContextOptions<DataContext> options) : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
    }
}
```

- In the `Startup.cs`, add the following service:

```c#
services.AddDbContext<DataContext>(opt =>
    {
        opt.UseSqlServer(Configuration.GetConnectionString("Default"));
    });
```

- and in the `appsettings.json`:

```json
"ConnectionStrings": {
    "Default": "Server=.\\SQLExpress; Database=db_dev; Integrated Security=SSPI;"
  }
```

- To add migration:

```dos
dotnet ef migrations add InitialModel -p Persistence/ -s Application/
```

- The first time, the command above won't run and you have to install `"Microsoft.EntityFrameworkCore.Design"` in the `Application` project. After installation, re-run the command above.
- A new migration will be created in the `Migrations` folder in the `Persistence` project.
- To apply the changes to the db:

```dos
dotnet ef database update -p Persistence/ -s Application/
```

## Seeding Database

- Create an empty migration by running the following command without changing the model:

```dos
dotnet ef migrations add SeedDatabase -p Persistence/ -s Application/
```

- In the `Up` method, write:

```c#
migrationBuilder.Sql("INSERT INTO Makes (Name) VALUES ('Make1')");
migrationBuilder.Sql("INSERT INTO Makes (Name) VALUES ('Make2')");
migrationBuilder.Sql("INSERT INTO Makes (Name) VALUES ('Make3')");

migrationBuilder.Sql("INSERT INTO Models (Name, MakeID) VALUES ('Make1-ModelA', (SELECT ID FROM Makes WHERE Name = 'Make1'))");
migrationBuilder.Sql("INSERT INTO Models (Name, MakeID) VALUES ('Make1-ModelB', (SELECT ID FROM Makes WHERE Name = 'Make1'))");
migrationBuilder.Sql("INSERT INTO Models (Name, MakeID) VALUES ('Make1-ModelC', (SELECT ID FROM Makes WHERE Name = 'Make1'))");

migrationBuilder.Sql("INSERT INTO Models (Name, MakeID) VALUES ('Make2-ModelA', (SELECT ID FROM Makes WHERE Name = 'Make2'))");
migrationBuilder.Sql("INSERT INTO Models (Name, MakeID) VALUES ('Make2-ModelB', (SELECT ID FROM Makes WHERE Name = 'Make2'))");
migrationBuilder.Sql("INSERT INTO Models (Name, MakeID) VALUES ('Make2-ModelC', (SELECT ID FROM Makes WHERE Name = 'Make2'))");

migrationBuilder.Sql("INSERT INTO Models (Name, MakeID) VALUES ('Make3-ModelA', (SELECT ID FROM Makes WHERE Name = 'Make3'))");
migrationBuilder.Sql("INSERT INTO Models (Name, MakeID) VALUES ('Make3-ModelB', (SELECT ID FROM Makes WHERE Name = 'Make3'))");
migrationBuilder.Sql("INSERT INTO Models (Name, MakeID) VALUES ('Make3-ModelC', (SELECT ID FROM Makes WHERE Name = 'Make3'))");
```

- In the `Down` method, write the opposite:

```c#
migrationBuilder.Sql("DELETE FROM Makes WHERE Name IN ('Make1', 'Make2', 'Make3')");
```

# Data Transfer Objects (Dtos)

Input and output of our controller actions should not be our object models. Because:

- `Firstly`, they are `implementation details` and we change them too often while there should be a contract for the API. For example, we might want nested objects from the API. But we know that in `EFCore` we can't have complex types that are not another table.
- `Secondly`, in the update process, a field that should not be updated, might be updated. For example, `LastUpdate` should be set on the server. Also, we don't want to expose some data to the client such as foreign keys.
- `Thirdly`, our model objects are connected to each other via nav props and create a loop when serializing to JSON.

## Creating Dtos

- So, we create a `Dtos` or `Resources` folder in the `Application` project.
- We copy our domain model props to this Dto class and get rid of at least one side of nav props and extra annotations. Be careful about the namespaces. Dto classes, should refer another Dto class not domain models.
- Also, if we want to reshape our data and introduce complex types (nested objects), we introduce a new `Dto` (e.g. `ContactDto`, or `ProfileDto`) in this folder which do not have a counterpart in the domain models.

## Separating Read and Save Dtos

- Often, we want more information from the Dto that is coming from the API, in comparison to the Dto that the user sends to the API.
- For example, we don't want the user to be able to set the `LastUpdate` but we want the user to read it. We also want to send a lot of pre-joined data to the user such as `Make`, `Model`, and `Feature`s.
- So we can have two separate Dtos: `VehicleDto` and `SaveVehicleDto`. If some properties just have to be set once and the user shouldn't edit them, we will even break `SaveVehicleDto` into two other Dtos: `EditVehicleDto` and `CreateVehicleDto`.

## Validation of Dtos

- For `VehicleDto` which is a `Read Dto`, we don't need any validation.
- A simpler form of validation (which we won't use in CRR project. In `CRR`, we will use `FluentValidation` package for validation.) is to use annotations on the properties in each `Dto` class in the `Application` project. In this way, because we used [ApiController] attribute, after the binding, because the ModelState prop of the current controller is not valid, it automatically responds with a bad request response.
- `Validation attributes`:

  - [Required]
  - [Required(ErrorMessage = "The field is required")]
  - [StringLength(100)]
  - [Range(0, 999.99)]
  - [RegularExpression(@"^\d{3}-\d{3}-\d{4}$")]
  - [Compare("Other Property")]
  - [Phone]
  - [EmailAddress]
  - [CreditCard]
  - [Url]

- `Custom validation attributes`:
- You can create such classes and put it in a folder called `CustomValidations` in the `Application` project.
- Use it like `[ClassicMovie(1960)]`.

```c#
public class ClassicMovieAttribute : ValidationAttribute
{
    public ClassicMovieAttribute(int year)
    {
        Year = year;
    }

    public int Year { get; }

    public string GetErrorMessage() =>
        $"Classic movies must have a release year no later than {Year}.";

    protected override ValidationResult IsValid(object value,
        ValidationContext validationContext)
    {
        var movie = (Movie)validationContext.ObjectInstance;
        var releaseYear = ((DateTime)value).Year;

        if (movie.Genre == Genre.Classic && releaseYear > Year)
        {
            return new ValidationResult(GetErrorMessage());
        }

        return ValidationResult.Success;
    }
}
```

- For simpler custom validations such as `MustBeInFuture` or `MustBeTrue`, we can use another overload of `IsValid` which accepts only the `value`.

## AutoMapper

- Install `AutoMapper.Extensions.Microsoft.DependencyInjection` in the `Application` project and add `services.AddAutoMapper(typeof(MappingProfile).Assembly);` in the Startup.
- Then, we have to create a `MappingProfile.cs` to define our maps (create that in Application/Mapping):

```c#
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<Make, MakeDto>();
        CreateMap<Model, IdNameDto>();
        CreateMap<Vehicle, VehicleDto>()
            .ForMember(vd => vd.Contact, opt => opt.MapFrom(v => new ContactDto { Name = v.ContactName, Email = v.ContactEmail, Phone = v.ContactPhone }))
            .ForMember(vd => vd.Features, opt => opt.MapFrom(v => v.Features.Select(vf => vf.FeatureId)));

        CreateMap<VehicleDto, Vehicle>()
            .ForMember(v => v.Id, opt => opt.Ignore()) // We don't want the Id to be set from the outside.
            .ForMember(v => v.ContactName, opt => opt.MapFrom(vd => vd.Contact.Name))
            .ForMember(v => v.ContactEmail, opt => opt.MapFrom(vd => vd.Contact.Email))
            .ForMember(v => v.ContactPhone, opt => opt.MapFrom(vd => vd.Contact.Phone))
            .ForMember(v => v.Features, opt => opt.Ignore())
            .AfterMap((vd, v) => // When the easy maps are done, we compare the two props that are collections.
            //One is ICollection<int> and the other is ICollection<VehicleFeature>.
            {
                // Remove unselected features
                var removedFeatures = v.Features
                    .Where(vf => !vd.Features.Contains(vf.FeatureId));
                var cloned = new List<VehicleFeature>(removedFeatures);

                foreach (var vehicleFeature in cloned)
                {
                    v.Features.Remove(vehicleFeature);
                }

                // Add new features
                var addedFeatures = vd.Features
                    .Where(id => !v.Features.Any(vf => vf.FeatureId == id)) //Instead of Any, you can write a Select followed by a Contains.
                    .Select(id => new VehicleFeature { FeatureId = id, VehicleId = v.Id });

                foreach (var vehicleFeature in addedFeatures)
                {
                    v.Features.Add(vehicleFeature);
                }
            });
    }
}
```

- Be careful that these maps are unidirectional, so if we want the other way around, we have to create another map.

# Designing Routes

For each model in our domain that has the following conditions:

- We want to expose any sort of CRUD operation on them. Here, we don't want to do any CRUD on `Model` (car model). We just seeded them.
- They have `NOT` been created because of a many-to-many relationship, either with attributes or without.

We create one controller/route (The name of the controller should be the `plural form of the domain model` + `Controller` and we use `RESTful` convention to design the actions/routes in each controller).

Exceptionally, for `User` model, we create another controller/route (apart from `UsersController`) called `ProfilesController` to separate the `all other operations` from auth operations such as `Login`, `Register`, and `Me`/`CurrentUser`.

```c#
//MakesController
HttpGet("api/makes")

//FeaturesController
HttpGet("api/features")

//VehiclesController
HttpGet("api/vehicles")
HttpPost("api/vehicles")
HttpGet("api/vehicles/{id}")
HttpPut("api/vehicles/{id}")
HttpDelete("api/vehicles/{id}")

//PhotosController: Because Photo model is a child (on the one side) in a one-to-many
// relationship (one vehicle has many photos), we design the routes like:
HttpGet("api/vehicles/{vehicleId}/photos")
HttpGet("api/vehicles/{vehicleId}/photos/{photoId}")
HttpPost("api/vehicles/{vehicleId}/photos")

//Alternative routes for PhotosController:
HttpGet("api/photos")     //In this case, we should get the vehicleId from query string
HttpGet("api/photos/{id}")//api/photos/4?vehicleId=2
HttpPost("api/photos")

//UsersController
[AllowAnonymous]
HttpPost("api/users/login")

[AllowAnonymous]
HttpPost("api/users/register")

HttpGet("api/users/me")

//ProfilesController
HttpGet("api/profiles/{username}")

HttpPut("api/profiles")
```

For the models that have the following conditions:

- We want to expose any sort of CRUD operation on them.
- They have been created because of a many-to-many relationship `with attributes` and they usually have their own name, like `Attendance`, `Enrollment`, `Follow`, `Book`, ...

Create actions/routes in one of its parent's controller. If one of the parent is `User` and the user can be inferred from HttpContext, create the routes in the other parent's controller.

In `ActivitiesController`:

```c#
HttpPost("api/activities/{id}/attend")   //To create an attendance
HttpDelete("api/activities/{id}/attend") //To delete an attendance
```

In `CoursesController`:

```c#
HttpPost("api/courses/{id}/enrol")   //To create an enrollment
HttpDelete("api/courses/{id}/enrol") //To delete an enrollment
```

Because `Follow` is a self-referencing many-to-many relationship and two sides are `User`, we have only one controller (`ProfilesController` not `UsersController`) to put the actions in, and the current user can be inferred from the HttpContext. But if we needed more data, like getting the list of followers of a user, we would use query strings:

```c#
HttpPost("api/profiles/{username}/follow")  //To follow a user
HttpDelete("api/profiles/{username}/follow")//To un-follow a user
HttpGet("api/profiles/{username}/follow")   //To get the list of followers or followings -> api/profiles/ben/follow?listof=followers
```

For the models that have the following conditions:

- We want to expose any sort of CRUD operation on them.
- They have been created because of a many-to-many relationship `without attributes` and they are just like tables with two-parts names, like `CourseTag`, `VehicleFeature`.

Do the CRUD operation (which is just adding or removing), via the parent. So they don't need their own controllers.

# Controllers

## Introduction

- Controller is a class that handles a `HTTP request`.
- Methods in a controller are called `action`s.
- An `API` controller is better to derive from `ControllerBase` class.
- A controller that returns view (e.g. `FallbackController` to serve a SPA) must derive from `Controller` class.
- The `Controller` class is derived from `ControllerBase` class.

## Route Attribute

- We use `Route` attribute above each controller class to prefix all the actions in that controller with that route:

```c#
[Route("api/[controller]")] //[controller] is automatically replaced by the current controller
public class VehiclesController : ControllerBase
```

- We can override this on any action, or even on the controller class if we have defined a `BaseController` class with `[Route("api/[controller]")]` attribute.

## Http Attributes

- You can also specify the route with or without any route parameters (they should be inside curly braces) in the parentheses in front of `Http` attributes:

```c#
[HttpGet]
[HttpGet("{id}")]
[HttpPost]
[HttpPut("{id}")]
[HttpDelete("{id}")]
[HttpPost("{id}/follow")]
```

## Binding Source Parameter Inference

When specifying the source of parameters in an action method, before the type, we put one of these to specify the source:

- [FromBody] Request body -> Used when the data is in the body of a `POST`, `PUT`, or, `DELETE` request.
- [FromForm] Form data in the request body -> Used when the data is coming from form (for example `file upload`)
- [FromHeader] Request header
- [FromQuery] Request query string parameter
- [FromRoute] Route data from the current request

```c#
[HttpPut("{id}")]
public IActionResult Update([FromRoute] string id, [FromBody] TodoItem item);
```

- It seems that even if we don't include [ApiController], the binding for route parameters and query strings (when they have the same name as the action parameters) is done automatically. It is called `The model binding system` which
  - Retrieves data from various sources such as route data, form fields, and query strings.
  - Provides the data to controllers and Razor pages in method parameters and public properties.
  - Converts string data to .NET types.

## Return Types

- For web API controller action return types, we have these two options:
  - `IActionResult` enables you to return a type deriving from ActionResult, like `return NotFound();` and `return Ok(vehicleDto);`.
  - `ActionResult<T>` enables you to return a type deriving from ActionResult or return a specific type, like `return NotFound();` and `return vehicleDto;`. SO this is better!

## ApiController Attribute

The `ApiController` attribute can be applied to a controller class to enable the following behaviors:

- It makes model validation errors automatically trigger an HTTP 400 response. So we `don't` have to write such code:

```c#
if (!ModelState.IsValid)
{
    return BadRequest(ModelState);
}
```

- Binding source parameter inference: So we don't need to define [FromBody], [FromQuery], [FromRoute],... attributes explicitly.

# Repository Pattern

Currently, we have two problems:

1. A query is repeated multiple places.
2. `Application` project is thightly-coupled to the `EF` and `Persistence` project (We have `using` statements). This cannot be solved only by changing folders. We want `Application` and `Persistence` projects (layers) to only depend on the `Core` layer.

The solution is to use `Repository Pattern`:

1. Create a folder `Interfaces` in the `Core` project to host `IUnitOfWork` and `IVehicleRepository`.
2. Create a concrete implementations of those in the `Persistence` project.
3. Register them as a service in `Startup`. We have three options (choose `AddScoped`):
   - `AddSingleton` just one instance of `VehicleRepository` will be created for the `app lifetime`. (router object is singleton in express framework. In C#, `HttpContextAccessor` should be registered as singleton to give access to something like `_httpContextAccessor.HttpContext.Request` property inside a service. It's only necessary to use IHttpContextAccessor when you need access to the HttpContext inside a service. `HttpContext` class encapsulates all HTTP-specific information about an individual HTTP request. `Request` property is of type `HttpRequest` Class which has properties such as `Scheme` or `Host`, ...)
   - `AddScoped` just one instance of `VehicleRepository` will be created for `each request`.
   - `AddTransient` just one instance of `VehicleRepository` will be created for `each injection` in the controller's ctor or any other operation class that receives an instance of `IVehicleRepository`.

- **Note 1** that a repository is actually a collection of `domain objects` in `memory`.
  So you cannot give repositories anything but domain models and we can't have repository of anything but domain models.
- **Note 2** each method in a repository must `not` return `IQueryable`. Only `IEnumerable` in case of `List` method.
- **Note 3** our `Application` is still dependent on `Persistence`, because we have to register the repositories in `Startup`. But our controllers are not dependent.
- **Note 4** 3-tier architecture has nothing do with this patten. 3-tier architectures physical distribution of a software like a `db server`, `application server`, and a `client` running on different machines. But `layer` is conceptual and is group of classes and packages who have one purpose projects in our solutions.

# Filtering, Sorting and Pagination

- Define an `IFilter` interface in the `Core` project (`Interfaces` folder).
- Define a `VehicleFilter` type which implements `IFilter` in the `Core` project (`Types` folder).
- Define an `VehicleFilterDto` class in the `Application` project (`Dtos` folder).
- Define an `Envelope<T>` type in the `Core` project (`Types` folder).
- Define an `EnvelopeDto<T>` class in the `Application` project (`Dtos` folder).
- Define a mapping in `MappingProfile`:

```c#
CreateMap(typeof(Envelope<>), typeof(EnvelopeDto<>));
CreateMap<VehicleFilterDto, VehicleFilter>();
```

- Extend `IQueryable` generic class in the `Persistence` project and add two methods of `ApplyOrdering` and `ApplyPaging` which both accept `this IQueryable<T> query` as the first parameter and return `IQueryable<T>`.
- Use these extensions methods in the `VehicleRepository` in a method.
- Use that method in `VehiclesController`.
- Now you can hit a route like `api/vehicles?sortBy=make&isSortDescending=true&pageSize=2&page=2`.

# File Upload

- Modify the db:
  - Add `Photo` entity in the `Core` project.
  - Add the corresponding prop to its parent (Vehicle).
  - Add `public DbSet<Photo> Photos {get; set;}` to `DataContext`.
  - Add a new migration.
- Create the Dto:
  - Add `PhotoDto` in the `Application` and get rid of foreign key.
  - Create mapping `CreateMap<Photo, PhotoDto>();`.
- Create the repository:
  - Create `IPhotoRepository` in the `Core` project and `PhotoRepository` in the `Persistence` project and a method `List` to list photos of each vehicle. You don't have to create a method for `Create`; because adding is done through the parent (a `vehicle` object, which is coming from vehicleRepository).
  - Register it in the `Startup`:

```c#
services.AddScoped<IPhotoRepository, PhotoRepository>();
```

- Strongly-type the `PhotoSettings`:
  - Create the `PhotoSettings` class in the `Types` folder of the `Application` project.
  - Add the settings in `appsettings.json` file under `PhotoSettings`.
  - Register it in the `Startup`:

```c#
services.Configure<PhotoSettings>(Configuration.GetSection("PhotoSettings"));
```

- Create `PhotoStorage` Service:
  - Create an `IPhotoStorage` interface in the `Core` project.
  ```c#
  public interface IPhotoStorage
  {
      Task<PhotoUploadResult> AddPhoto(IFormFile file);
      string DeletePhoto(string publicId);
  }
  ```
  - Create `PhotoUploadResult` type in the `Core` project.
  ```c#
  public class PhotoUploadResult
    {
        public string PublicId { get; set; }
        public string Url { get; set; }
    }
  ```
  - Create an `Infrastructure` project:
  ```dos
  dotnet new classlib -n Infrastructure
  dotnet sln add Infrastructure/
  cd Infrastructure
  dotnet add reference ../Core  ::Make sure that the target framework is netcoreapp3.1.
  cd ../Application
  dotnet add reference ../Infrastructure
  ```
  - Create `FileSystemPhotoStorage` class which implements `IPhotoStorage` in the `Infrastructure` project.
  - Register it in the `Startup`:
  ```c#
  services.AddTransient<IPhotoStorage, FileSystemPhotoStorage>();
  ```
  - Create `wwwroot` in the root of `Application` project.
  - In the `Startup`, before `useRouting`, add:
  ```c#
  app.UseDefaultFiles();
  app.UseStaticFiles();
  ```
- Create `PhotosController` but you have to specify the `[Route("api/vehicles")]` above class.

# Authentication and Authorization

- We don't use cookies, because it only works for the browser and not mobile clients. Here, the client should store the `JWT` in the `local storage` and sends it back to the server.

## Add User Entity

- In the `Core` project, add `User.cs`, install `Microsoft.AspNetCore.Identity.EntityFrameworkCore`, and restore. Also, add nav-prop to the `Vehicle` class.
- Add the props that do not already exist in the `IdentityUser` class.
- In the `DataContext` class, change `DbContext` to `IdentityDbContext<AppUser>` and add `base.OnModelCreating(builder);` in the `OnModelCreating` method.
- Add a new migration by `dotnet ef migrations add "AddedIdentity" -p Persistence -s API/`.

## Configure Identity in the application

To enable the application to create, manage, and login users via a UserManager and a signInManager services:

- In the `Application` project, install `Microsoft.AspNetCore.Identity.UI`, and restore.
- in the `Startup` class, add:

```c#
services.AddDefaultIdentity<User>().AddEntityFrameworkStores<DataContext>();
```

## Dtos

- Because it sends back all the info about the user and we don't want that, create a `UserDto.cs`, `ProfileDto`, `LoginDto`, `SaveProfileDto` and `SaveUserDto` in the `Application` project.

## JWT

We are going to make `Infrastructure` project to generate a jwt for us:

- Create `IJwtGenerator` interface inside `Core` project.
- Create the implementation of the above interface (`JwtGenerator`) in the `Infrastructure` project (For that, install `System.IdentityModel.Tokens.Jwt` in the `Infrastructure` project).
- Add `services.AddScoped<IJwtGenerator, JwtGenerator>();` in the `Startup.cs`.
- Add `"TokenKey": "super secret key",` to `appsettings.json`.

## Retrieve Username from a jwt

- Create `IUserAccessor` interface inside `Core` project.
- Create the implementation of the above interface (`UserAccessor`) in the `Infrastructure` project.
- Add `services.AddScoped<IUserAccessor, UserAccessor>();` in the `Startup.cs`.

## UserRepository

- Create the repository:
  - Create `IUserRepository` in the `Core` project and `UserRepository` in the `Persistence` project and a method `Details` to retrieve one user and his/her vehicles (this is used to calculate the number of his/her vehicles in the MappingProfile).
  - Register it in the `Startup`:

```c#
services.AddScoped<IUserRepository, UserRepository>();
```

## Controllers

- Create `UsersController` and `ProfilesController`.
- Also, modify the `VehiclesController` route to add `UserId` to the newly-created vehicle and check the current user to be the owner of the vehicle for deleting and editing. We could use `Auth Policies` for that.

## Securing API

- Add `Microsoft.AspNetCore.Authentication.JwtBearer` package to the `Application` project.
- Add the following in the `Startup.cs`:

```c#
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(opt =>
    {
        opt.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(Configuration["TokenKey"])),
            ValidateAudience = false,
            ValidateIssuer = false,
            ValidateLifetime = true,
            ClockSkew = TimeSpan.Zero
        };
    });
```

- Add `UseAuthentication` middleware in the `Configure` method in this order:

```
UseRouting -> UseAuthentication -> UseAuthorization -> UseEndpoints
```

- Use `[Authorize]` attribute on any method (route) that you want to protect.
- Now the consumer has to provide `Authorization` header with `Bearer ...` for each request to access it.

## Authorization policy

Add this option to the `AddControllers` method in the `Startup`:

```c#
services.AddControllers(opt =>
{
    var policy = new AuthorizationPolicyBuilder().RequireAuthenticatedUser().Build();
    opt.Filters.Add(new AuthorizeFilter(policy));
});
```

Now, all the routes requires authorization. So add `[AllowAnonymous]` to the `Login` and `Register` routes in the `UsersController` class and remove `[Authorize]` from previous methods.
