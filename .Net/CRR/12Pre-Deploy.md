# Deploy

### Client

We want our API to serve the static client app (not a good approach of course).

- Add `"postbuild": "move build ../API/wwwroot",` to the `scripts` section of `package.json` file. So every time a build is created, it automatically appears in wwwroot folder.
- Then, we build the project. In the client directory:

```dos
npm run build
```

- Now the `wwwroot` folder is created in the `API` project with the build files.
- In the `Startup`, add

```c#
app.UseDefaultFiles();
app.UseStaticFiles();
```

before `useRouting`.

- To map the routes that are unknown to our API to `Index.html`, add the followings to the routing:

```c#
endpoints.MapFallbackToController("Index", "Fallback");
```

- Now, create the `Fallback` controller (which derives from `Controller` and have `View` support, unlike the `ControllerBase`) with the `Index` method in it.

### Database

- Add `Microsoft.EntityFrameworkCore.SqlServer` to the `Persistence` project and remove `Microsoft.EntityFrameworkCore.Sqlite`.
- In `appsettings.Development.json`:

```json
"ConnectionStrings": {
    "Default": "Server=.\\SQLExpress; Database=crr_dev; Integrated Security=SSPI;"
  },
```

and in `appsettings.json` to:

```json
"ConnectionStrings": {
    "Default": "Server=.\\SQLExpress; Database=crr_prod; Integrated Security=SSPI;"
},
```

- Now, change `UseSqlite` to `UseSqlServer` in the `Startup`.
- Change the environment of the project (`API`) in the `launchSettings.json` to `Production` or `Development`.
- Delete all the previous migrations!
- Run `dotnet ef migrations add SqlServerMigration -p Persistence/ -s API/` to generate the migration to `SQL Server`.
- Then, run the `API` project either by `dotnet run` or `dotnet watch run`.
