# Photo Upload

### Adding Photo Entity

- Add `Photo` entity in the `Domain` project.
- Add the corresponding prop to its parent (`AppUser`).
- Add `public DbSet<Photo> Photos {get; set;}` to `DataContext`.
- Add a new migration.

### Cloudinary settings

- Sign-up and verify the email
- In the `API/` path:

```dos
dotnet user-secrets set "Cloudinary:CloudName" "dgflfmqgd"
dotnet user-secrets set "Cloudinary:ApiKey" "657623884722349"
dotnet user-secrets set "Cloudinary:ApiSecret" "oXZVgso1sBI1sESEYv3lqGReDPg"
```

- Create the `CloudinarySettings.cs` in the `Infrastructure` project to strongly-typed the above magic strings.
- Add `services.Configure<CloudinarySettings>(Configuration.GetSection("Cloudinary"));` in the `Startup`.
- Install `CloudinaryDotNet` in the `Infrastructure` project.

### IPhotoStorage and PhotoStorage

- Add `IPhotoStorage` in the `Application/Interfaces`
- Add `PhotoStorage` in the `Infrastructure/Photos`
- Add `services.AddScoped<IPhotoStorage, PhotoStorage>();` in the `Startup.cs`

### Adding the `Add` command handler

Although, adding operation is a command but because we cannot create the `PublicId` and `Url` in the client, we have to return the `Photo` from it (like what we did with the register operation).

### Adding the Photo API Controller
