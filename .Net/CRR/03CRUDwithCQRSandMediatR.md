# CRUD using the CQRS and Mediator pattern

### Another method of seeding db

In the previous method for seeding data, (using `HasData` in `OnModelCreating` method), we must provide the id. But in this method, we don't have to do it. Create `Seed.cs` in the `Persistence` namespace:

```C#
if (!context.Activities.Any())
            {
                var activities = new List<Activity>
                {
                    new Activity
                    {
                        Title = "Past Activity 1",
                        Date = DateTime.Now.AddMonths(-2),
                        Description = "Activity 2 months ago",
                        Category = "drinks",
                        City = "London",
                        Venue = "Pub",
                    },
                    //...
                };

                context.Activities.AddRange(activities);
                context.SaveChanges();
            }
```

and then add `Seed.SeedData(context);` in the `Main` method of `Program.cs`.

### Resetting the database

```dos
dotnet ef database drop -p Persistence/ -s API/
```

Choose `y` and then `dotnet watch run` in the `API` project.

### Command-Query Responsibility Separation (CQRS)

- Command

  - Does something
  - Modifies the state of the db
  - It should not return a value

- Query
  - Answers a question
  - Does not Modify the state of the db
  - It should return a value

This architecture is particularly beneficial, if the write-database and the read-database are different (by using an eventual consistency approach [of course it has the risk of the read-database not being up-to-date]):

![](/md/cqrs.jpg)

### Mediator pattern

Just like the `Repository pattern`, this pattern is designed to decouple the API controllers from the persistence layer. Each controller has a mediator of type `IMediator` which has a `Send` method and depending on the type of the object passed to it, the `Handle` method in the `Handler` class will be invoked.
In the NuGet package manager, search for `Mediatr`, add `MediatR.Extensions.Microsoft.DependencyInjection` to the `Application` project and `restore`.
Then, inside dependency injection container (`ConfigureServices` method), add the following and pass the assembly that the `Handler`s are located at:

```c#
services.AddMediatR(typeof(List.Handler).Assembly);
```

### Create code snippets

- First create a snippet file for `Query Handler` and `Command Handler` by pressing `File>Preferences>User Snippets` and then `New Snippets file for ...`. Type `handler` for the name.
- Search for `Snippet Generator VSCode` on Google and paste the code that you want to extract an snippet out of. After declaring the placeholders and add each snippet to the handler.code-snippets, you should end-up having:

```json
{
  "Query Handler": {
    "prefix": "qhandler",
    "body": [
      "public class Query : IRequest<${1:example}> { }",
      "",
      "        public class Handler : IRequestHandler<Query, ${1:example}>",
      "        {",
      "            private readonly DataContext _context;",
      "            public Handler(DataContext context)",
      "            {",
      "                _context = context;",
      "            }",
      "",
      "            public async Task<${1:example}> Handle(Query request, CancellationToken cancellationToken)",
      "            {",
      "                //handler logic goes here",
      "",
      "                return ;",
      "            }",
      "        }"
    ],
    "description": "Query Handler"
  },
  "Command Handler": {
    "prefix": "chandler",
    "body": [
      "public class Command : IRequest",
      "        {",
      "        }",
      "",
      "        public class Handler : IRequestHandler<Command>",
      "        {",
      "            private readonly DataContext _context;",
      "            public Handler(DataContext context)",
      "            {",
      "                _context = context;",
      "            }",
      "",
      "            public async Task<Unit> Handle(Command request, CancellationToken cancellationToken)",
      "            {",
      "                //handler logic goes here             ",
      "",
      "                var success = await _context.SaveChangesAsync() > 0;",
      "",
      "                if (success) return Unit.Value;",
      "",
      "                throw new Exception(\"Problem saving changes\");",
      "            }",
      "        }"
    ],
    "description": "Command Handler"
  }
}
```

### CQRS + MediatR Pattern

1. For each resources that you want to create a controller for, make a directory in the `Application` project with the plural form of that resource.
1. Also, create a controller for each resource

```c#
[Route("api/[controller]")]
[ApiController]
public class CustomController : ControllerBase
{
    private readonly IMediator _mediator;
    public CustomController(IMediator mediator)
    {
        _mediator = mediator;
    }
}
```

3. For each operation for any specific resource, create a c# class with the name of the operation. Note that the name of method is not important in the API controllers because the route matching is done by the route, route parameters and the type of request.
1. Type the `qhandler` or `chandler` depending of the type of the operation and in case of `qhandler`, specify the return-type.
1. Complete the `Query` or `Command` class. This is the class that an instance of will be passed to the mediator. So, based on what you want to get in the handler, enter the properties.
1. Complete the handler logic.
1. Add the action in the related controller:

```c#
[Http???]
public async Task<ActionResult<ReturnType>> Operation(params)
{
    return await _mediator.Send(a query or command object);
}
```

### Add support for CORS

Now to add Cross Origin Resource Sharing (`CORS`) support, in the `ConfigureServices` method in `Startup` add the followings:

```c#
services.AddCors(opt =>
{
    opt.AddPolicy("CorsPolicy", policy =>
    {
        policy.AllowAnyHeader().AllowAnyMethod().WithOrigins("http://localhost:3000");
    });
});
```

and in the `Configure` method in `Startup` add the following middleware:

```c#
app.UseCors("CorsPolicy");
```
