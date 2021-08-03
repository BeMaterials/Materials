# API Skeleton

### Architecture

![](/md/arch.jpg)

### Create the solution and projects

Getting help:

```Dos
dotnet -h
dotnet new -h
...
```

In the new/empty directory to create the solution file:

```Dos
dotnet new sln
```

To create the projects in the solution:

```Dos
dotnet new classlib -n Domain
dotnet new classlib -n Application
dotnet new classlib -n Persistence
dotnet new webapi -n API
```

### Adding projects to the solution and Adding dependencies of each project

Now, we have to reference the `.csproj` to add each project:

```Dos
dotnet sln add Domain/
dotnet sln add Application/
dotnet sln add Persistence/
dotnet sln add API/
```

To check:

```Dos
dotnet sln list
```

Now, adding the dependencies:

```Dos
cd Application
dotnet add reference ../Domain/
dotnet add reference ../Persistence/

cd ../API
dotnet add reference ../Application/

cd ../Persistence
dotnet add reference ../Domain/
```

In each project folder, there are two folders `bin` and `obj` which will be created each time the project is built and we don't want to see them, so we exclue them in VSCode: `File>Preferences>Settings`.

Comment out the `app.UseHttpsRedirection();` middleware in `Startup.cs` and remove `https://localhost:5001;` from `API/Properties/launchSettings.json` in the `API:applicationUrl` section.

### Running the API project

Create the `ValuesController.cs` and then:

```Dos
dotnet run -p API/
```

Now you should get this message in the console:

> Now listening on: http://localhost:5000

Then, go to [http://localhost:5000/api/values](http://localhost:5000/api/values)

### Creating the database

Install

- `Microsoft.EntityFrameworkCore` and
- `Microsoft.EntityFrameworkCore.Sqlite`
  by using `NuGet Package Manager`. For both, choose the `version 3.1.3` and the `Persistence` project.
- Now, in the command line run `dotnet restore`.

Change `Startup.cs` by adding:

```C#
services.AddDbContext<DataContext>(opt =>
{
    opt.UseSqlite(Configuration.GetConnectionString("Default"));
});
```

and add the following to the `appSettings.json`:

```json
"ConnectionStrings":{
    "Default": "Data source=crr.db"
  },
```

Now create a new entity in the Domain `Value.cs` with two auto-implemented props `Id` and `Name`. Then, create the `DataContext.cs` in the `Persistence`.

Now to create a migration, we need to install dotnet-ef command line tool:

```Dos
dotnet tool install --global dotnet-ef
```

First shut down the app by `ctrl + c` to create the migration (-p to show EF how to find the context and -s to give EF the startup project and connection string):

```Dos
dotnet ef migrations add InitialCreate -p Persistence/ -s API/
```

For the above command to work, you need to add `Microsoft.EntityFrameworkCore.Design` to the `API` project. After adding, re-run the above command. By doing so, the `Migrations` folder will be created in the `Persistence` project.

You can update the database to reflect the migrations by:

```Dos
dotnet ef database update
```

**But** we modify the `Main` method of the `API` project in such a way that whenever the app starts up, it checks to see that the db exists (otherwise it will create it) and runs the latest migration.

```C#
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
```

Then, run the API again by `dotnet run -p API/`. Now, the file `crr.db` is created inside the API project.

**Important note** by `dotnet watch run`, if we change any file, the app will automatically restart. <del>`dotnet watch run -p API/`</del> does not work. It only works in the context of startup project (`API` project here).

Now, by `ctrl + shift + p` type `SQLite` and select `Open Database`. Then, choose `crr.db`. In the left panel, you can see two tables: _Values_ and _\_EFMigrationHistory_.

### Seeding data

Override the `OnModelCreating` method in the `DataContext`:

```C#
protected override void OnModelCreating(ModelBuilder builder)
{
    builder.Entity<Value>()
        .HasData(
            new Value {Id = 1, Name = "Value 101"},
            new Value {Id = 2, Name = "Value 102"},
            new Value {Id = 3, Name = "Value 103"}
        );
}
```

The `HasData()` method is used to seed the table. Now, create a new migration (make sure that the app is shut down):

```Dos
dotnet ef migrations add SeedValues -p Persistence/ -s API/
```

Finally, `dotnet watch run` (from API folder).
