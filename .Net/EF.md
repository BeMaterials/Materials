# Introduction

## Setup

- In the root folder:

```dos
dotnet new sln

dotnet new classlib -n Domain
dotnet new classlib -n Persistence
dotnet new console -n Application

dotnet sln add Domain/
dotnet sln add Persistence/
dotnet sln add Application/

cd Persistence
dotnet add reference ../Domain/

cd ../Application
dotnet add reference ../Domain/
dotnet add reference ../Persistence/
```

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

## DbContext

- DbContext is an abstract over the database and provides a simple API to load data from it and save in it.
- Create `DataContext` (or `ApplicationDataContext`) in the root of `Persistence` project:

```c#
public class DataContext : DbContext //It should derive from DbContext in EntityFramework
{
    //Here we expose DbSets...

    //For the API apps, we add this constructor:
    public DataContext(DbContextOptions<DataContext> options) : base(options)
    {
    }

    //For the consol apps, we override OnConfiguring method and specify the connection string in it:
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(@"Server=.\SQLExpress; Database=my_db; Integrated Security=SSPI;"); //SQLExpress is our server. my_db is the name of db to be created. SSPI means use Windows authentication.
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        //If we want EF, uses configuration instead of convention, one way is to use this method.
    }
}
```

### Console apps

In the `program.cs`:

```c#
using (var context = new DataContext())
{
    // do stuff
}
```

### API apps

in the `Startup.cs`, add the following service:

```c#
services.AddDbContext<DataContext>(opt =>
    {
        opt.UseSqlServer(Configuration.GetConnectionString("Default"));
    });
```

and in the `appsettings.json`:

```json
"ConnectionStrings": {
    "Default": "Server=.\\SQLExpress; Database=crr_prod; Integrated Security=SSPI;"
  }
```

For using `DataContext`, just inject it in the ctor of a repository in the `Repository pattern` apps:

```c#
private readonly DataContext _context;

public CustomRepository(DataContext context)
{
    _context = context;
}

```

or in the case of `CQRS and Mediator pattern` apps, in the `Handler` class:

```c#
private readonly DataContext _context;

public Handler(DataContext context)
{
    _context = context;
}
```

## Domain Models

- These are entities of your Domain which are POCOs (Plain old CLR objects).
- For each entity create a model class in the `Domain` project and expose a DbSet in the DataContext. Each will be a table in the db.
- The props in each domain model should be either regular C# types (int, Guid, float, string, DateTime, enum, ...) or if it is a custom type (that we have defined) it should `correspond to a table` in our domain.
- For `value type`s such as `int` and `DateTime`, if you want them to be `nullable` use `int?` and `DateTime?`
- EF is convention-based:
  - Each prop with the name of `Id` will be an `identity` and a `primary key`.
  - Each prop with the name of `AnotherModelId` along with another prop with the name of `AnotherModel` will be a `foreign key`.
  - EF will create index for both `pk`s and `fk`s.
- Nav-props:
  - On the parent side (one in the one-to-many relationships) which is usually ICollection (It can be List, IList,...) is `not mandatory`. If we include them, we should initialize them in the ctor to prevent `NullReferenceException`.
  - On the child side (many in the one-to-many relationships) which is part of the `foreign key` that I talked about above is `mandatory`.

```c#
public class Author
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Course> Courses { get; set; } //Optional nav-prop. By this, we can easily access the courses of each author with . notation.

    public Author()
    {
        Courses = new Collection<Course>();
    }
}
```

```c#
public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public float FullPrice { get; set; }
    public CourseLevelId CourseLevelId { get; set; }  //CourseLevelId is an enum.
    public CourseLevel CourseLevel { get; set; }  //Required nav-prop.
    public int AuthorId { get; set; }
    public Author Author { get; set; } //Required nav-prop.
    public ICollection<CourseTag> Tags { get; set; } //Optional nav-prop.
    public Course()
    {
        Tags = new Collection<CourseTag>();
    }
}
```

```c#
public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<CourseTag> Courses { get; set; }
    public Tag()
    {
        Courses = new Collection<CourseTag>();
    }
}
```

The definition of the enum used in `Course` model is:

```c#
public enum CourseLevelId
{
    Beginner = 0,
    Intermediate = 1,
    Advanced = 2
}
```

- For each `many-to-many` relationships `with attributes`, create a model class in the domain and expose a DbSet in the context; because they are independent entities. You have to add a composite key definition in `OnModelCreating` method because EFCore does not automatically infer it like the case with just one `fk`:

```c#
builder.Entity<Attendee>().HasKey(at => new { at.AppUserId, at.ActivityId });
```

- For each `many-to-many` relationships `without attributes` create a model class in the domain but do not expose a DbSet in the context; because you probably don't want to use them like `_context.CourseTags`. But EFCore will create a table in the migrations (they are called `link tables`).

```c#
public class CourseTag
{
    public int CourseId { get; set; }
    public Course Course { get; set; }
    public int TagId { get; set; }
    public Tag Tag { get; set; }
}
```

You have to add a composite key in `OnModelCreating` method in this case too:

```c#
modelBuilder.Entity<CourseTag>().HasKey(ct => new { ct.CourseId, ct.TagId });
```

- For the entities `with static records`, create a model class in the domain with two props of `Id` (which should be an enum) and `Name` (of type string). But do not expose a DbSet in the context; because you probably don't want to use it like `_context.CourseLevels`. But EFCore will create a table in the migrations (they are called `lookup tables`).

```c#
public class CourseLevel
{
    public CourseLevelId Id { get; set; }
    public string Name { get; set; }
    public ICollection<Course> Courses { get; set; }
    public CourseLevel()
    {
        Courses = new Collection<Course>();
    }
}
```

You have to seed lookup tables in `OnModelCreating` method:

```c#
modelBuilder.Entity<CourseLevel>().HasData(
    Enum.GetValues(typeof(CourseLevelId)).Cast<CourseLevelId>() //This will return an IEnumerable<CourseLevelId> of all the values of an enum.
        .Select(courseLevelId => new CourseLevel()
        {
            Id = courseLevelId,
            Name = courseLevelId.ToString()
        })
);
```

After creating all the models, the `DataContext` in the `Persistence` project should be like:

```c#
public class DataContext : DbContext
{
    public DbSet<Author> Authors { get; set; }
    public DbSet<Course> Courses { get; set; }
    public DbSet<Tag> Tags { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(@"Server=.\SQLExpress; Database=my_db; Integrated Security=SSPI;");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<CourseTag>().HasKey(ct =>
            new { ct.CourseId, ct.TagId });

        modelBuilder.Entity<CourseLevel>().HasData(
            Enum.GetValues(typeof(CourseLevelId))
                .Cast<CourseLevelId>()
                .Select(courseLevelId => new CourseLevel()
                {
                    Id = courseLevelId,
                    Name = courseLevelId.ToString()
                })
        );
    }
}
```

## Migrations

### Add migrations

- From now on, for each changes in the model just add a migration with the ChangeName:

```dos
dotnet ef migrations add InitialModel -p Persistence/ -s Application/
```

- The first time, the command above won't run and you have to install `"Microsoft.EntityFrameworkCore.Design"` in the `Application` project. After installation, re-run the command above.
- A new migration will be created in the `Migrations` folder in the `Persistence` project. It has a class with the name of our migration, prefixed by the date. There are two methods of Up (which will execute the changes on the db) and Down (which will undo the changes from the db) for each migration.
- _Important_: If you rename a column, there will be an `ADD` and a `DROP` statement in the migration and we lose data. In this case use `Sql("SQL statements goes here");` to copy the old column to the new column between that two statements. Make sure that you do the opposite in the `Down` method:

```c#
migrationBuilder.Sql(@"
    UPDATE Authors
    SET FirstName = Name;
");
```

- With code-first approach, you have full versioning of your db.
- You really should treat migrations just like git commits.
  - So, for small changes create a migration. In this way you can give it a meaningful name.
  - Never delete a migration and just create another one.
- EFCore only allow one pending migration at a time.

### Applying changes to the db or reverting changes

- To apply the changes to the db:

```dos
dotnet ef database update -p Persistence/ -s Application/
```

- To revert the changes to an older migration:

```dos
dotnet ef database update TargetMigration -p Persistence/ -s Application/
```

### Removing migrations

- To remove the last migration that hasn't been applied yet:

```dos
dotnet ef migrations remove -p Persistence/ -s Application/
```

- To remove the last migration that has been applied and there is no migration before it (not recommended):

```dos
dotnet ef database update 0 -p Persistence/ -s Application/
dotnet ef migrations remove -p Persistence/ -s Application/
```

- To remove the migrations that have been applied up to a point in time (not recommended):

```dos
dotnet ef database update TargetMigration -p Persistence/ -s Application/
dotnet ef migrations remove -p Persistence/ -s Application/
```

### Working on a feature branch

To get the db that corresponds to the code-base that you are working on, you have two options:

- If you don't need the current data in the database:
  1. Checkout to the feature branch.
  2. Change the name of the db in the connection string to something else.
  3. Run `dotnet ef database update -p Persistence/ -s Application/`.
  4. You will have an empty new database with the correct db structure.
- If you need the current data in the database:
  1. Run `dotnet ef database update TargetMigration -p Persistence/ -s Application/`.
  1. Checkout to the feature branch.
  1. Work on the new feature.
  1. Checkout to the master branch.
  1. Run `dotnet ef database update -p Persistence/ -s Application/`.

### Seeding data

- Create an empty migration (without changing the model, run `...migrations add PopulateAuthorTable...` command).
- In the `Up` method, write:

```c#
migrationBuilder.Sql(@"
    INSERT INTO authors VALUES ('John'), ('Ben'), ('Adam');
");
```

- In the `Down` method, write the opposite (Only remove these authors! Do not wipe out the entire table.).

# Configuration over Convention

Just use one of `Data Annotations` or `Fluent API` methods and do not mix both. `Fluent API` approach is better and more professional than the `Data Annotations` approach.

## Data Annotations

- It is decorating props in the domain models with certain attributes.
- Make sure that the TargetFramework `<TargetFramework>netcoreapp3.1</TargetFramework>` is `netcoreapp3.1` in all projects.
- The annotations are defined in `System.ComponentModel.DataAnnotations` and `System.ComponentModel.DataAnnotations.Schema` namespaces.

```c#
[Table("tbl_categories")] //Changes the table name in the db. The default is the plural form of the class name.
public class Category
{
    [Key] //Sets this as pk. The default is to use Id or CategoryId as pk.
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    //DatabaseGeneratedOption.Identity -> The corresponding column will be Auto Incremented.
    //DatabaseGeneratedOption.Computed -> The value of the corresponding column will be calculated in the db.
    public string ISBN { get; set; }
    [Column("sName", TypeName = "varchar(200)")]
    //Changes the name and/or type of a column.
    //The default for the name is the prop name.
    //The default for the type: string -> nvarchar(MAX). int -> int.
    [Required]
    //Makes the column NOT NULL. The default: every nullable prop is nullable in the db as well.
    //Note that you can set this prop to null in the code but will throw when persisting to db.
    public string Name { get; set; }
    [MaxLength(50)] //Sets limit for the string length. The default is nvarchar(MAX).
    public string Type { get; set; }
}
```

## Fluent API

- It is overriding `OnModelCreating` in the `DataContext` class.
- Suppose we have these domain models:

```c#
public class Book
{
    public string Title { get; set; }
    public string ISBN { get; set; }
    public DateTime? CreatedAt { get; set; }
    public int WriterId { get; set; }   //Because this is not AuthorId, we have to define fk in the Fluent API and it won't be generated automatically.
    public Author Writer { get; set; }  //one-to-many relationship -> Book is child.
    public Caption Caption { get; set; }//one-to-zer/one relationship -> Book is parent.
    public Cover Cover { get; set; }    //one-to-one relationship -> Book is parent.

}

public class Caption
{
    public string BookId { get; set; } //The fk is pk too (ISBN).
    public Book Book { get; set; }
    public string Body { get; set; }

}

public class Cover
{
    public string BookId { get; set; } //The fk is pk too (ISBN).
    public Book Book { get; set; }
    public string Text { get; set; }
}
```

We have to add `Books` prop to `Author` model too. The `Fluent API` configuration will be as follows:

```c#
modelBuilder.Entity<Book>()
    .ToTable("tbl_book"); //Changes the table name in the db. The default is the plural form of the class name.

modelBuilder.Entity<Book>()
    .HasKey(b => b.ISBN); //Sets this as pk. The default is to use Id or BookId as pk.

modelBuilder.Entity<Caption>()
    .HasKey(c => c.BookId);

modelBuilder.Entity<Cover>()
    .HasKey(c => c.BookId);

modelBuilder.Entity<CourseTag>()
    .HasKey(ct => new { ct.CourseId, ct.TagId }); //We can't set composite pk with data annotations

modelBuilder.Entity<Book>()
    .Property(b => b.Title).HasColumnName("Name"); //The default is the prop name.

modelBuilder.Entity<Book>()
    .Property(b => b.Title).HasColumnType("varchar(200)"); //The default for: string -> nvarchar(MAX). int -> int.

modelBuilder.Entity<Book>()
    .Property(b => b.ISBN).ValueGeneratedNever(); //Equivalent to [DatabaseGenerated(DatabaseGeneratedOption.None)]

modelBuilder.Entity<Book>()
    .Property(b => b.Title).HasDefaultValue("Unknown_Title"); //Default value -> fixed

modelBuilder.Entity<Book>()
    .Property(b => b.CreatedAt).HasDefaultValueSql("getdate()"); //Default value -> computed

modelBuilder.Entity<Caption>()
    .Property(c => c.Body).HasMaxLength(200); //Sets limit for the string length. The default is nvarchar(MAX).

modelBuilder.Entity<Cover>()
    .Property(c => c.Text).IsRequired(); //Makes the column NOT NULL. The default: every nullable prop is nullable in the db as well.

modelBuilder.Entity<Book>() //one-to-many relationship. Started from Book.
    .HasOne(b => b.Writer)
    .WithMany(a => a.Books)
    .HasForeignKey(b => b.WriterId)
    .OnDelete(DeleteBehavior.Restrict);

// modelBuilder.Entity<Author>()  //one-to-many relationship. Started from Author. (Pick this one or the one above. Not both!)
//     .HasMany(a => a.Books)
//     .WithOne(b => b.Writer)
//     .HasForeignKey(b => b.WriterId)
//     .OnDelete(DeleteBehavior.Restrict);

modelBuilder.Entity<Book>()  //one-to-zero/one relationship.
    .HasOne(b => b.Caption)
    .WithOne(c => c.Book)
    .HasForeignKey<Caption>(c => c.BookId); //When configuring the foreign key you need to specify the dependent entity type.

modelBuilder.Entity<Book>()  //one-to-one relationship.
    .HasOne(b => b.Cover)
    .WithOne(c => c.Book)
    .HasForeignKey<Cover>(c => c.BookId); //When configuring the foreign key you need to specify the dependent entity type.
```

### Best practice

- Create a folder called `EntityConfigurations` in the `Persistence` project. For each entity, create a class called `CustomConfiguration` which derives from `IEntityTypeConfiguration<Custom>`:

```c#
class BookConfiguration : IEntityTypeConfiguration<Book>
{
    public void Configure(EntityTypeBuilder<Book> builder)
    {
        //First ToTable
        builder.ToTable("tbl_book");

        //Second, HasKey
        builder.HasKey(b => b.ISBN);

        //Third, Properties in alphabetical order
        builder.Property(b => b.CreatedAt).HasDefaultValueSql("getdate()"); ;
        //...

        //Fourth, Relationships
        builder.HasOne(b => b.Caption)
                .WithOne(c => c.Book)
                .HasForeignKey<Caption>(c => c.BookId);
        //...

    }
}
```

- In the `OnModelCreating` method:

```c#
modelBuilder.ApplyConfiguration(new BookConfiguration());
```

# Query with Linq

- With `Linq`, you don't have to write SQL queries/stored procedures for `simple` queries.
- For `hard` queries, it is better to use `vanilla SQL`.

## Queries that return IQueryable<>

- These queries do not execute right away. They will be executed in two ways:
  - They appear as the second arg of a `foreach` loop.
  - One of `ToList()`, `ToArray()`, or `ToDictionary()` methods is invoked on them. These methods bring the values to the `memory`.
- The good thing about these queries is that, because they don't execute right a way, we can extend these queries later.

### Common queries

```c#
var courses = context.Courses
    .Where(c => c.Title.Contains("c#"))
    .OrderBy(c => c.Title);

foreach (var course in courses)
{
    Console.WriteLine(course.Title);
}

var advancedCourses = context.Courses
    .Where(c => c.CourseLevelId == CourseLevelId.Advanced);

var courses1 = context.Courses
    .OrderBy(c => c.Title)
    .ThenBy(c => c.CourseLevelId);

var courses2 = context.Courses
    .OrderByDescending(c => c.Title);

var coursesWithAuthor = context.Courses
    .Select(c => new { Name = c.Title, Author = c.Author.Name }); //Anonymous object

var tags = context.Courses
    .SelectMany(c => c.Tags);
//SelectMany flattens the result.
//Returns IQueryable<CourseTag>.
//If Select was used, it would return IQueryable<ICollection<CourseTag>>.

foreach (var tag in tags)
{
    Console.WriteLine(tag.TagId); //Note that, we don't have access to tag.Tag.Name, because we didn't include it in the query.
}

var uniqueTagIds = context.Courses
    .SelectMany(c => c.Tags)
    .Select(ct => ct.TagId)
    .Distinct();

foreach (var tagId in uniqueTagIds)
{
    Console.WriteLine(tagId);
}

var groups = context.Courses
    .GroupBy(c => c.AuthorId) //It doesn't work with nav-props
    .Select(g => new { Author = g.Key, CourseCount = g.Count() });
    //Usage of an aggregate function right after calling GroupBy is very common

foreach (var group in groups)
{
    Console.WriteLine($"{group.Author} {group.CourseCount}");
}
```

### Joins

- One strength of `EFCore` is that with the `nav-props`, we don't have to write `JOIN`. But if for a reason we had to omit the nav-prop, for example in the `Course` class we had only one prop for `AuthorId` and we had no `Author`, we have to use `JOIN`:

#### Inner join:

```c#
var coursesWithAuthor = context.Authors.Join(
    context.Courses,
    a => a.Id, //Left key
    c => c.AuthorId, //Right key
    (author, course) => new { Author = author.Name, Course = course.Title }
);

foreach (var courseWithAuthor in coursesWithAuthor)
{
    Console.WriteLine($"{courseWithAuthor.Course} by {courseWithAuthor.Author}");
}
```

#### Cross join

```c#
var combos = context.Authors
    .SelectMany(a => context.Courses, (author, course) => new { Author = author.Name, Course = course.Title });

foreach (var combo in combos)
{
    Console.WriteLine($"{combo.Course} by {combo.Author}");
}
```

### Paging queries

```c#
var courses = context.Courses.Skip(5).Take(5);

foreach (var course in courses)
{
    Console.WriteLine(course.Title);
}
```

## Queries that return one value

- These queries execute right a way.

```c#
var course1 = context.Courses.First();
var course2 = context.Courses.First(c => c.FullPrice > 24);
var course3 = context.Courses.FirstOrDefault(c => c.FullPrice > 30);

var course4 = context.Courses.Single(c => c.FullPrice > 24);
var course5 = context.Courses.SingleOrDefault(c => c.FullPrice > 30);

var course6 = context.Courses.Find(15); //This method is only used on primary key. If There are multiple pks, use ','.
// Equal to = context.Courses.Single(c => c.Id == 15);

var allInLevel0 = context.Courses.All(c => c.CourseLevelId == CourseLevelId.Beginner); //False
var anyInLevel0 = context.Courses.Any(c => c.CourseLevelId == CourseLevelId.Beginner); //True

var count = context.Courses.Count();
var maxPrice = context.Courses.Max(c => c.FullPrice);
var minPrice = context.Courses.Min(c => c.FullPrice);
var avgPrice = context.Courses.Average(c => c.FullPrice);
var sum = context.Courses.Sum(c => c.FullPrice);
```

# Loading Related Data

- After loading data using the above queries, if we want to access the `nav-props` of the loaded data, we have to load that related data too.
- We have three options:

## Lazy loading

- This means loading the related data on-demand.
- To `activate` lazy loading:
  - Instal `"Microsoft.EntityFrameworkCore.Proxies` in the `Persistence` project.
  - For `API apps`, in the `Startup.cs`, in `services.AddDbContext<DataContext>(opt => {`, add this option too: `opt.UseLazyLoadingProxies();`.
  - For `Console apps`, in the `DataContext.cs`, in `OnConfiguring` method, add this option too: `optionsBuilder.UseLazyLoadingProxies();`.
  - Make all nav props `virtual`.
- On the `many side` of a `one-to-many` relationship, it is ok to use `lazy loading`:

```c#
var author = context.Authors.Find(3); // 1 call to the db

foreach (var course in author.Courses) // 1 call to the db
{
    Console.WriteLine($"{course.Title} by {course.Author.Name}");
}
```

- When this approach is used on the `one side` of a `one-to-many` relationship, it can result into `N + 1` issue:

```c#
var courses = context.Courses.ToList(); // 1 call to the db

foreach (var course in courses)
{
    Console.WriteLine($"{course.Title} by {course.Author.Name}"); // 1 call to the db for each iteration to load Author
}
```

- We usually don't use `lazy loading` in `web apps` because of this problem.

## Eager loading

- If `lazy loading` hasn't been activated, `nav props` will be `null` or `empty collection`.
- So you have to `Include` the related data in the query.
- Best place to use `Eager loading` is the `one side` of a `one-to-many` relationship:

```c#
var courses = context.Courses
    .Include(c => c.Author) //Defined in: using Microsoft.EntityFrameworkCore;
    .ToList(); // 1 call to the db

foreach (var course in courses)
{
    Console.WriteLine($"{course.Title} by {course.Author.Name}"); // No call to the db
}
```

- It can also be used in the `many side` of a `one-to-many` relationship:

```c#
var author = context.Authors
    .Include(a => a.Courses)
    .SingleOrDefault(a => a.Id == 3); //After Include, you can't use Find.

foreach (var course in author.Courses) // No call to the db. Already populated.
{
    Console.WriteLine($"{course.Title} by {course.Author.Name}");
}
```

### Multi-level eager loading

```c#
var author = context.Authors
    .Include(a => a.Courses)
        .ThenInclude(c => c.Tags)
            .ThenInclude(ct => ct.Tag)
    .Include(a => a.Books)
        .ThenInclude(b => b.Cover)
    .SingleOrDefault(a => a.Id == 3);

foreach (var course in author.Courses)
{
    Console.WriteLine($"{course.Title}:");
    foreach (var tag in course.Tags)
    {
        Console.WriteLine(tag.Tag.Name);
    }
}
```

## Explicit loading

- Instead of one gigantic eager query with many `Include`s that can be complicated, we can have multiple `explicit queries`.
- The `Load` method is defined in `using Microsoft.EntityFrameworkCore;`, invoked on an `IQueryable` and brings the value to the memory.

```c#
var author = context.Authors.Find(3); //1 call to db

context.Courses.Where(c => c.AuthorId == 3).Load(); //1 call to db

foreach (var course in author.Courses) //Because the courses for author with the id of 3 is in the memory, we can use it with no problem.
{
    Console.WriteLine($"{course.Title} by {course.Author.Name}");
}
```

- An `advantage` of explicit loading over eager loading is that we can have filter conditions instead of loading all related data:

```c#
var author = context.Authors.Find(3);

context.Courses.Where(c => c.AuthorId == 3 && c.FullPrice > 15).Load();

foreach (var course in author.Courses)
{
    Console.WriteLine($"{course.Title} by {course.Author.Name}");
}
```

- Another interesting example:

```c#
var authors = context.Authors.Where(a => a.Name.StartsWith("J")).ToList();
var authorIds = authors.Select(a => a.Id);

context.Courses.Where(c => authorIds.Contains(c.AuthorId) && c.FullPrice < 20).Load();

foreach (var author in authors)
{
    foreach (var course in author.Courses)
    {
        Console.WriteLine($"{course.Title} ${course.FullPrice} by {author.Name}");
    }
}
```

# Add, Update, and Delete

- There is a `ChangeTracker` prop in the `context` instance, which is responsible for tracking changes in the `DataContext` for `Add`, `Update`, and `Delete`.
- When the `SaveChanges` method is called on the context, the queries will be built, applied to the db, and restore the status to `Unchanged`.

## Add a record

```c#
context.Courses.Add(new Course
{
    Title = "Ben's course",
    FullPrice = 15.5f,
    CourseLevelId = CourseLevelId.Intermediate,
    AuthorId = 1
});

context.SaveChanges();
```

- You have to use only one `Add` call to the context for saving related data. All the relations are in the memory:

```c#
var author = new Author { Name = "Alex" };

context.Authors.Add(author);

author.Courses.Add(new Course
{
    Title = "Alex's course",
    FullPrice = 5.5f,
    CourseLevelId = CourseLevelId.Intermediate,
});

context.SaveChanges();
```

## Update a record

```c#
var course = context.Courses.Find(15);

course.Description = "Better desc...";

context.SaveChanges();
```

## Delete a record

```c#
var course = context.Courses.Find(15);

context.Remove(course);

context.SaveChanges();
```

- If `Delete Rule` for the `fk` is set to `No Action`, we will get a `DbUpdateException` when executing this query:

```c#
var author = context.Authors.Find(2);

context.Remove(author);

context.SaveChanges();
```

So we re-write it to:

```c#
var author = context.Authors
    .Include(a => a.Courses)
    .SingleOrDefault(a => a.Id == 2);

context.Courses.RemoveRange(author.Courses); //RemoveRange accepts an IEnumerable<>

context.Remove(author);

context.SaveChanges();
```

- Use `isDeleted` field as a `Bit` instead of deleting.
