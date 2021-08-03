# Introduction

## Schema

- You have to `define` a schema.
- That schema `in` something called `GQL`: GraphQL Query Language.
- You can create a schema with `Schema.For(@"GQL")`.
- There are `three types of schema`:
  1. `Types`
  2. `Query` (what we can query for)
  3. `Mutation` (what we can change)
- For example, we can query for hello and that the response is of type string:

```c#
var schema = Schema.For(@"
    type Query {
        hello: String
    }
    ", _ =>
      {
        _.Types.Include<Query>();
      });
```

- `_.Types.Include<Query>();` means we are pointing out a class named `Query` that will handle all incoming queries.

## Resolvers

- A resolver is simply a `method` that `knows` what to do with an `incoming request` that can be of type `Query` or `Mutation`.
- We will define `all query resolvers` in a class called `Query` and `all mutation resolvers` in a class called `Mutation`.
- The decorator `GraphQLMetadata` helps us to map what's written in the Schema to a method, a _resolver_.
- What goes as the GraphQLMetadata argument, is just the query or mutation identifier (without its potential parameters).
- If a query or mutation receives a parameter, create an input parameter with the same parameter name and type (type `ID` in GQL corresponds to `int` in C#).

```c#
[GraphQLMetadata("hello")]
public string GetHello()
{
    return "World";
}
```

## GraphiQL

- Is a `visual environment` that allows you to run queries.

## How to query

A typical query would be like:

```gql
{
  users(id: 1) {
    name
    created
  }
}
```

What we are saying above is that we want to query for users with the id of 1 and we want the columns name and created. The response is like:

```json
{
  "data": {
    "users": [
      {
        "name": "chris",
        "created": "2018-01-01"
      }
    ]
  }
}
```

# Web API using GraphQL

## Setup

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
- In `Application` project, install the `GraphQL` (version 2.4.0) and `GraphiQL` packages and in `Persistence` project install the `Microsoft.EntityFrameworkCore`, and `Microsoft.EntityFrameworkCore.SqlServer` packages by using NuGet Package Manager.
- Then,

```dos
cd ..
dotnet restore
```

## Create domain models

Author model:

```c#
using System.Collections.Generic;

namespace Core.Domain
{
    public class Author
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public List<Book> Books { get; set; }

        public Author()
        {
            Books = new List<Book>();
        }
    }
}
```

Book model:

```c#
namespace Core.Domain
{
    public class Book
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Genre { get; set; }
        public bool IsPublished { get; set; }
        public int AuthorId { get; set; }
        public Author Author { get; set; }
    }
}
```

## Adding database

- Create `DataContext` in the root of `Persistence` project:

```c#
using Core.Domain;
using Microsoft.EntityFrameworkCore;

namespace Persistence
{
    public class DataContext : DbContext
    {
        public DbSet<Author> Authors { get; set; }
        public DbSet<Book> Books { get; set; }

        public DataContext(DbContextOptions<DataContext> options) : base(options)
        {
        }
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

- Modify the Main method of the Application project in such a way that whenever the app starts up, it checks to see that the db exists (otherwise it will create it) and runs the latest migration:

```c#
public static void Main(string[] args)
{
    var host = CreateHostBuilder(args).Build();

    using (var scope = host.Services.CreateScope())
    {
        var services = scope.ServiceProvider;
        try

        {
            var context = services.GetRequiredService<DataContext>();
            context.Database.Migrate();
        }
        catch (Exception ex)
        {
            var logger = services.GetRequiredService<ILogger<Program>>();
            logger.LogError(ex, "An error occurred during migration");
        }
    }

    host.Run();
}
```

## Seeding Database

- Create `Seed.cs` in the Persistence namespace:

```c#
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Core.Domain;

namespace Persistence
{
    public class Seed
    {
        public static async Task SeedData(DataContext context)
        {
            if (!context.Authors.Any())
            {
                var author = new Author
                {
                    Name = "Stephen King"
                };

                author.Books.AddRange(new List<Book>
                {
                    new Book
                        {
                            Title = "IT",
                            IsPublished = true,
                            Genre = "Mystery"
                        },
                    new Book
                        {
                            Title = "The Langoleers",
                            IsPublished = true,
                            Genre = "Mystery"
                        }
                }
                );

                context.Authors.Add(author);

                await context.SaveChangesAsync();
            }
        }
    }
}
```

- Add `Seed.SeedData(context).Wait();` in the Main method of Program.cs.

## Define the schema

Create a file `MySchema.cs` in `Application/GraphQL`:

```c#
using GraphQL.Types;

namespace Application.GraphQL
{
    public class MySchema
    {
        private readonly ISchema _schema;
        public ISchema GraphQLSchema => _schema;

        public MySchema()
        {
            _schema = Schema.For(@"
          type Book {
            id: ID
            title: String,
            genre: String,
            published: Date,
            Author: Author
          }

          type Author {
            id: ID,
            name: String,
            books: [Book]
          }

          type Query {
              books: [Book]
              author(id: ID): Author,
              authors: [Author]
              hello: String
          }

          type Mutation {
            addAuthor(name: String): Author
          }
      ", _ =>
      {
          _.Types.Include<Query>();
          _.Types.Include<Mutation>();
      });
        }
    }
}
```

- `_.Types.Include<Query>();` and `_.Types.Include<Mutation>();` mean that we are pointing out a class named `Query` and a class named `Mutation` that will handle all incoming queries and mutations, respectively. We have to create these classes.

## Resolvers

### Query resolvers

```c#
using System.Collections.Generic;
using System.Threading.Tasks;
using Core.Domain;
using GraphQL;
using Microsoft.EntityFrameworkCore;
using Persistence;

namespace Application.GraphQL
{
    [GraphQLMetadata("Query")]
    public class Query
    {
        private readonly DataContext _context;

        public Query(DataContext context)
        {
            _context = context;
        }

        [GraphQLMetadata("books")]
        public async Task<IEnumerable<Book>> GetBooks()
        {
            return await _context.Books
                .Include(b => b.Author)
                .ToListAsync();
        }

        [GraphQLMetadata("authors")]
        public async Task<IEnumerable<Author>> GetAuthors()
        {
            return await _context.Authors
                .Include(a => a.Books)
                .ToListAsync();
        }

        [GraphQLMetadata("author")]
        public async Task<Author> GetAuthor(int id)
        {
            return await _context.Authors
                .Include(a => a.Books)
                .SingleOrDefaultAsync(a => a.Id == id);
        }

        [GraphQLMetadata("hello")]
        public string GetHello()
        {
            return "World";
        }
    }
}
```

### Mutation resolvers

```c#
using System.Threading.Tasks;
using Core.Domain;
using GraphQL;
using Persistence;

namespace Application.GraphQL
{
    [GraphQLMetadata("Mutation")]
    public class Mutation
    {
        private readonly DataContext _context;

        public Mutation(DataContext context)
        {
            _context = context;
        }

        [GraphQLMetadata("addAuthor")]
        public async Task<Author> AddAuthor(string name)
        {
            var author = new Author
            {
                Name = name
            };

            _context.Authors.Add(author);
            await _context.SaveChangesAsync();

            return author;
        }
    }
}
```

## Adding GraphQLQuery type

```c#
using Newtonsoft.Json.Linq;

namespace Application.GraphQL
{
    public class GraphQLQuery
    {
        public string OperationName { get; set; }
        public string NamedQuery { get; set; }
        public string Query { get; set; }
        public JObject Variables { get; set; }
    }
}
```

## Adding the GraphQL route

When it comes to GraphQL the whole point is to only have one route `/graphql` and for a negotiation to happen between frontend and backend about what content should be returned back.

- In `Startup.cs` and in the method Configure() we need to add `app.UseGraphiQl("/graphiql");` before `app.UseEndpoints`,
- Create `GraphqlController`:

```c#
using System.Threading.Tasks;
using Application.GraphQL;
using GraphQL;
using Microsoft.AspNetCore.Mvc;

namespace Application.Controllers
{
    [Route("graphql")]
    [ApiController]
    public class GraphqlController : ControllerBase
    {
        [HttpPost]
        public async Task<ActionResult> Post([FromBody] GraphQLQuery query)
        {
            var schema = new MySchema();
            var inputs = query.Variables.ToInputs();

            var result = await new DocumentExecuter().ExecuteAsync(_ =>
            {
                _.Schema = schema.GraphQLSchema;
                _.Query = query.Query;
                _.OperationName = query.OperationName;
                _.Inputs = inputs;
            });

            if (result.Errors?.Count > 0)
            {
                return BadRequest();
            }

            return Ok(result);
        }
    }
}
```

## How to query

A query would be like:

```gql
{
  books {
    title
    genre
    published
    author {
      name
    }
  }
}
```

with the corresponding answer:

```json
{
  "data": {
    "books": [
      {
        "title": "IT",
        "genre": "Horror",
        "published": "1994",
        "author": {
          "name": "Stephen King"
        }
      }
    ]
  }
}
```
