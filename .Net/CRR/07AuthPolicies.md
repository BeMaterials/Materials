# Auth Policies

We could write our authorization logic inside each `Handle` method to see a user owns an activity so he is able to delete it. BUT, using policies is more flexible.

- So, we create `IsHostRequirement.cs` class inside `Infrastructure` project.
- In the `Startup` add:

```c#
services.AddAuthorization(opt =>
{
    opt.AddPolicy("IsActivityHost", policy =>
    {
        policy.Requirements.Add(new IsHostRequirement());
    });
});

services.AddTransient<IAuthorizationHandler, IsHostRequirementHandler>();
```

- Add this attribute `[Authorize(Policy = "IsActivityHost")]` to each route to apply the policy.
