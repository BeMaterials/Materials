### Self-referencing many-to-many relationship

For two reasons, we have to add the following two blocks:

1. The name of the nav property is not the same as the model because it is self-referencing and we cannot have two properties with the same name of AppUser.
2. We want to change the default behavior of OnDelete for this relationship.

```c#
builder.Entity<Follow>()
    .HasOne(f => f.Follower)
    .WithMany(u => u.Followings)
    .HasForeignKey(f => f.FollowerId)
    .OnDelete(DeleteBehavior.Restrict);

builder.Entity<Follow>()
    .HasOne(f => f.Following)
    .WithMany(u => u.Followers)
    .HasForeignKey(f => f.FollowingId)
    .OnDelete(DeleteBehavior.Restrict);
```

or, equivalently (I prefer this one:):

```c#
builder.Entity<AppUser>()
    .HasMany(u=>u.Followers)
    .WithOne(f=>f.Following)
    .HasForeignKey(f=>f.FollowingId)
    .OnDelete(DeleteBehavior.Restrict);

builder.Entity<AppUser>()
    .HasMany(u=>u.Followings)
    .WithOne(f=>f.Follower)
    .HasForeignKey(f=>f.FollowerId)
    .OnDelete(DeleteBehavior.Restrict);
```
