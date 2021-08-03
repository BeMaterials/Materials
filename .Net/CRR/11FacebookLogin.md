# Facebook Login

First create a new app in [here](developers.facebook.com) to get an App Id and a secret.

### API

```dos
dotnet user-secrets set Authentication:Facebook:AppId 1401085816740456
dotnet user-secrets set Authentication:Facebook:AppSecret secretkjhkjhsdf234
```

- To strongly type these settings, create a type in the `Infrastructure` project called `FacebookAppSettings` with two props.
- Add `services.Configure<FacebookAppSettings>(Configuration.GetSection("Authentication:Facebook"));` in the `Startup`.
- Create `IFacebookAccessor` in the `Application/Interfaces`.
- Create the type `FacebookUserInfo` (in the `Application/Types`) which is expected from `FacebookLogin` method in that interface.
- Create the concrete implementation (`FacebookAccessor`).
- Register in the `Startup`: `services.AddScoped<IFacebookAccessor, FacebookAccessor>();`.
- Create the `FacebookLogin` handler.
- Add the endpoint in `UserController`.

### Client

- Add `npm i react-facebook-login @types/react-facebook-login`
-
