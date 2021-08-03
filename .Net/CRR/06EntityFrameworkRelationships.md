# Entity Framework Relations

### Many-to-many relations

The `AppUser` and `Activity` entities have many-to-many relation. Because EFCore does not support these kind of relationships, we have to create another entity called `Attendee`. In this way, we can also add another props to this new entity:

```c#
public class Attendee
{
    public string AppUserId { get; set; }
    public AppUser AppUser { get; set; }
    public Guid ActivityId { get; set; }
    public Activity Activity { get; set; }
    public DateTime DateJoined { get; set; }
    public bool IsHost { get; set; }
}
```

We also add `Nav props` to the corresponding parents (`Activity` and `AppUser`):

```c#
public ICollection<Attendee> Attendees { get; set; }
```

Then, we need to add the `public DbSet<Attendee> Attendees { get; set; }` to the `DataContext` and since EFCore does not support `Data Annotations` for `Composite Keys`, we have to add:

```c#
builder.Entity<Attendee>().HasKey(at => new { at.AppUserId, at.ActivityId });
```

Then, create the migration with `dotnet ef migrations add "AttendeeAdded" -p Persistence/ -s API/`

### Eager loading

In the `List` and `Details` operations, modify the `Handle` method to include the related data:

```c#
var activities = await _context.Activities
    .Include(a => a.Attendees)
    .ThenInclude(at => at.AppUser)
    .ToListAsync();
```

```c#
var activity = await _context.Activities
    .Include(a => a.Attendees)
    .ThenInclude(at => at.AppUser)
    .SingleOrDefaultAsync(a => a.Id == request.Id); //We can't use FindAsync after Include
```

Here, because we haven't used `DTOs` yet, the application goes to a self-referencing loop state.

### DTOs

Create `ActivityDTO.cs` and `AttendeeDto.cs` inside `Application/Activities`:

```c#
public class ActivityDto
{
    public Guid Id { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public string Category { get; set; }
    public DateTime Date { get; set; }
    public string City { get; set; }
    public string Venue { get; set; }
    public ICollection<AttendeeDto> Attendees { get; set; }
}
```

```c#
public class AttendeeDto
{
    public string Username { get; set; }
    public string DisplayName { get; set; }
    public string Image { get; set; }
    public bool IsHost { get; set; }
}
```

Now, we have to re-write our operations so that they receive a DTO and return one.
For that, we need to install `AutoMapper.Extensions.Microsoft.DependencyInjection` in the `Application` project and add `services.AddAutoMapper(typeof(List.Handler).Assembly);` in the `Startup`.
Then, we have to create a `MappingProfile` to define our maps (create that in `Application/Activities`):

```c#
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<Activity, ActivityDto>();
        CreateMap<Attendee, AttendeeDto>()
            .ForMember(ad => ad.Username, opt => opt.MapFrom(a => a.AppUser.UserName))
            .ForMember(ad => ad.DisplayName, opt => opt.MapFrom(a => a.AppUser.DisplayName));
    }
}
```

Now, we change our operations. And we have to change our API controllers as well.
