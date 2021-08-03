# Basics

## Setup

```dos
dotnet new sln
dotnet new console -n Playground
dotnet sln add Playground/
```

- There are different kind of projects: classlib, console, api, ...
- Every project has its own namespace.
- A classlib will be compiled to a `dll` assembly.
- A console app will be compiled to an `exe` assembly.
- If you want to add a `reference` to another project (usually the other project is classlib):

```dos
cd Playground
dotnet add reference ../AnotherProject/
```

- Note that, you can't have circular referencing.
- To run a specific project:

```dos
dotnet run -p Playground
```

- To build a specific project:

```dos
dotnet build Playground/
```

- To build all projects in this solution:

```dos
dotnet build
```

## Snippets and shortcuts

### Snippets

- `cw` -> `Console.WriteLine();`
- `ctor` -> ...
- `prop` -> ...

### Shortcuts

- `F5` -> Run in debug mode
- `shift + F5` -> Stop debugging
- `F9` -> Toggle breakpoint
- `F10` -> step-over
- `F11` -> step-into a method
- `shift + F11`-> step-out of a method
- `F12` -> Object browser. Explore definition of a type
- `shift + F12` -> Shows references
- `ctrl + .` -> Many things: Move to another file, Using...

## Comments

- `//` inline comments
- `/* */` block comments

## Console

```c#
Console.Write("Say: "); //cursor remains at the same line
Console.WriteLine("Hi");//cursor goes to the next line

var str = Console.ReadLine(); // returns a string

Console.ForegroundColor = ConsoleColor.Red; //Change output color
Console.WriteLine("In red");
```

## Variables and constants

```c#
int number;
int number2 = 1; //you can initialize just after declaration
int @int; //if you want to use a reserved keyword as variable

const float Pi = 3.14; //in case of constants, you have to provide a value
```

## Primitive types

```c#
byte b = 100;      //System.Byte   -> 0-255           1Byte
short s = 100;     //System.Int16  -> +/- 32767       2Byte
int i = 100;       //System.Int32  -> +/- 2.1B        4Byte
long l = 100;      //System.Int64  -> +/- ...         8Byte

float f = 1.1;     //System.Single  -> +/- 3.4x10^38  4Byte
double d = 1.1;    //System.Double  -> +/- ...        8Byte
decimal dec = 1.1; //System.Decimal -> +/- 7.9x10^28  16Byte

char c = 'A';      //System.Char    -> Unicode Char   2Byte

bool isTrue = true;//System.Boolean -> true/false     1Byte
```

## Strings

- In `.Net`, the class `String` is the `string` type.
- You have to enclose it in " ".
- They are `reference type`.
- They are `immutable`. Once you create them, you cannot change them. All methods, return a new value. Being immutable has nothing to do with being reference type (some value types are immutable too).

```c#
string name = "Ben";
name[0] = 'J'; //won't compile. strings are immutable.
```

- The length of a string can be accessed by `Length` prop: `name.Length`
- Access each character in a string:

```c#
string name = "Ben";
var firstChar = name[0]; //of type char
```

- String from an array:

```c#
var arr = new int[] { 1, 2, 3 };
var list = string.Join(",", arr);
Console.WriteLine(list); //1,2,3
```

- `Format` string:

```c#
Console.WriteLine($"{byte.MinValue} - {byte.MaxValue}");
```

- Escape characters and verbatim strings:

```c#
Console.WriteLine("\tc:\\projects\\whatever\n\t\"Finished\"");
Console.WriteLine(@"        c:\projects\whatever");
```

- Showing currencies

```c#
var price = 2.95;
Console.WriteLine(price.ToString("C")); //$2.95
Console.WriteLine(price.ToString("C0"));//$3
```

- To check that if a string is empty:

```c#
Console.WriteLine(string.IsNullOrWhiteSpace("Ben")); //False
Console.WriteLine(string.IsNullOrWhiteSpace(null));  //True
Console.WriteLine(string.IsNullOrWhiteSpace(""));    //True
Console.WriteLine(string.IsNullOrWhiteSpace("    "));//True
```

### Some string methods:

```c#
var name = "Ben Abe  ";
Console.WriteLine(name.IndexOf(' '));                    //3
Console.WriteLine(name.LastIndexOf('e'));                //6
Console.WriteLine(name.StartsWith("Abe"));               //False
Console.WriteLine(name.Contains("en"));                  //True
Console.WriteLine(name.Trim());                          //"Ben Abe"
Console.WriteLine(name.ToUpper());                       //"BEN ABE  "
Console.WriteLine(name.Substring(0, name.IndexOf(' '))); //"Ben"
Console.WriteLine(name.Substring(name.IndexOf(' ') + 1));//"Abe  "
Console.WriteLine(name.Split(' ')[0]);                   //"Ben"
Console.WriteLine(name.Split(' ')[1]);                   //"Abe  "
Console.WriteLine(name.Replace("Ben", "John"));        //"John Abe  "
```

## Default value for each data type

With `default` operator, you can return the default value of each type.

```c#
Console.WriteLine(default(int));
```

- byte, short, int, long = 0
- float, double, decimal = 0.0
- char = ''
- bool = false
- DataTime = 01/01/0001
- string = null
- any objects = null

## Var keyword

- The compiler will infer the type of variable.
- _Limitation_: all the whole numbers will be converted to `int`.

```c#
var n1 = 1.2f; //float
var n2 = 1.2; //double is default
var n3 = 1.2m; //decimal
```

## Operators

```c#
int a = 1;
int b = a++; //a=2, b=1
int b = ++a; //a=2, b=2
```

All the operators are like `js`, except there is no `===` here.

```c#
int a = 10;
int b = 3;
Console.WriteLine(a / b); //3
Console.WriteLine((float)a / b); //3.3333333
```

## Overflowing

```c#
byte number = 255;
number++;
Console.WriteLine(number); //0
```

To combat, use `checked` which throws an exception:

```c#
checked
{
    byte number = 255;
    number++;
}
```

## Type conversions

### Implicit:

```c#
byte b = 1;
int i = b; //No Problem!
```

### Explicit:

```c#
int i = 1;
byte b = (byte)i; //if no cast, won't compile
```

### Possible problem with explicit conversion:

```c#
int i = 1000;
byte b = (byte)i;
Console.WriteLine(b); //232
```

### Non-compatible types:

All primitive types have `parse` method which takes a string and spits the target type.

```c#
string s = "1";
int i = int.Parse(s); //1
int j = Convert.ToInt32(s); //1
```

if `s="10000000000"`, `OverflowException`.

## Arrays

- Three ways of defining arrays:

```c#
var arr1 = new int[3];
arr1[0] =1;
arr1[1] =2;
arr1[2] =3;

var arr2 = new int[] { 1, 2, 3 };

int[] arr3 = { 1, 2, 3 };
```

_Note_: If a member of an array is not provided with a value, the default value of that data type will be used.

- Arrays from string:

```c#
var numbersString = "1,2,3,4";
Console.WriteLine(numbersString.Split(',')[0]);//1
```

### Multi-dimensional arrays

```c#
var matrix = new int[2, 3] { { 1, 2, 3 }, { 4, 5, 6 } };
Console.WriteLine(matrix[0, 1]); //2
```

### Jagged arrays (Array of arrays)

```c#
var arr = new int[2][] {
    new int[] { 1 },
    new int[] { 2, 3, 4 }
    };
Console.WriteLine(arr[1][2]); //4
```

### Array props and methods

```c#
var arr1 = new int[] { 1, 2, 3 };
var arr2 = new int[] { 4, 6, 5 };

Console.WriteLine(arr1.Length); //3

Array.Clear(arr1, 0, 2); //The first two members will be set to default values
Console.WriteLine(arr1[0]); //0

Console.WriteLine(Array.IndexOf(arr2, 5)); //2

Array.Reverse(arr2);
Console.WriteLine(arr2[0]); //5

Array.Sort(arr2);
Console.WriteLine(arr2[0]); //4
```

## Enums

- When we have some related constants, we use enums.
- Because they are new types, they have to be defined inside `ns` level.
- If you want to get the numeric value, you have to cast it to `(int)`.
- On the contrary, if you want to convert a number to enum, cast it to `(EnumName)`.

```c#
public enum ShippingMethods : byte //Optional. The default is int.
{
    Regular = 1, //if we omit '=1', it starts at 0
    Express = 2
}
```

and in the `Main` method:

```c#
var method = ShippingMethods.Express;

Console.WriteLine(method == ShippingMethods.Regular); //False
Console.WriteLine(method.ToString() == "Express"); //True

Console.WriteLine(method); //Express. Because 'cw' automatically invokes 'ToString()'
Console.WriteLine((int)method); //2
```

### Parsing a string to enum

```c#
var method = (ShippingMethods)Enum.Parse(typeof(ShippingMethods), "Regular");
```

## Intro to classes

- **very important**: The fields and props (and also local variables) can be defined as System data types (int, string, Guid, DateTime, delegates (event EventHandler, Func<T1, T2, ...>, Action<T1, T2, ...>, ...) ...) or the ones that we create. But we cannot nest them like typescript and we have to create another type for each level.

```c#
public class Person //public is the `access modifier` and Person is the `identifier`
{
    public string Name; //Instance field
    public static int Count = 0; //Static field. We don't have to initialize it in case of 0.

    public Person() //Constructor: the best place to initialize fields (we didn't do that here!)
    {
        Count++;
    }

    public void Introduce(string to = "stranger") //Instance method. Here, we also used default value for parameter (Optional Arguments)
    {
        Console.WriteLine($"Hi {to}, I am {Name}");
    }

    public void SayBye() => Console.WriteLine("Bye!"); //If a method has only one statement, it can be written like this: expression bodied method.

    public static Person Parse(string str) //A good example of a static method
    {
        var person = new Person();
        person.Name = str;
        return person;
    }

    public static int Add(int a, int b) => a + b; //Another Static method. If a method has just one return, we can write it like this: expression bodied method.

    public static int Multiply(int a, int b) //Another Static method.
    {
        var result = 0;
        for (int i = 0; i < a; i++)
        {
            result = Add(result, b); //In a static method, you can only call another static method
        }

        return result;
    }

    public static int Formula(int a, int b)
    {
        return a + 2 * b;
    }
}
class Program
{
    static void Main(string[] args)
    {
        // Static fields and methods are accessible through the class itself.
        Console.WriteLine(Person.Add(2, 2));           //4
        Console.WriteLine(Person.Multiply(7, 8));      //56
        Console.WriteLine(Person.Formula(b: 4, a: 1)); //9: Named Arguments
        Console.WriteLine(Person.Count);               //0

        var person = new Person();                     //An object or an instance
        person.Name = "John";
        person.Introduce();                            //Hi stranger, I am John
        person.Introduce("Tom");                       //Hi Tom, I am John
        person.SayBye();                               //Bye!

        Console.WriteLine(Person.Count);               //1

        var person2 = new Person() { Name = "Ben" };   //Object initialization syntax
        var person3 = new Person { Name = "Nas" };     // Or equivalently

        var person4 = Person.Parse("Tom");
    }
}
```

## Structs

- Structs are `almost exactly` like classes, BUT they are `value types`.
- They are usually used to store `light-weight` objects:

```c#
public struct RgbColor
{
    public byte Red;
    public byte Green;
    public byte Blue;
    public RgbColor(byte red, byte green, byte blue)
    {
        Red = red;
        Green = green;
        Blue = blue;
    }
}
```

## Data types

- Structs (Value types)

  - Primitive types
  - Custom structs
  - **Note 1:** Memory allocated in `stack`.
  - **Note 2:** When they become out of scope, they are destroyed.

- Classes (Reference types)

  - Arrays
  - Strings
  - Custom classes
  - **Note 1:** We have to allocate memory with `new`.
  - **Note 2:** Memory allocated in `heap`.
  - **Note 3:** Address of the object stored in `stack`.
  - **Note 4:** When they become out of scope, they stay for a while and then garbage-collected by `CLR`.

## Boxing and Unboxing

### Boxing

The process of converting a `value type` to a `reference type`.

```c#
Object obj = 10;
```

The value of `10` is wrapped and stored in `heap`.

### Unboxing

The process of converting a `reference type` to a `value type`.

```c#
Object obj = 10;
var number = (int)obj;
```

These two processes have performance penalty and needs to be avoided. So for example we don't use `ArrayList` to store `int` or `DateTime`.

## Loops

- We have `for`, `while`, `do-while` and `foreach` loops.
- In `foreach`, we can have enumerable types like, `Array`, `string`, or `List`.
- `break;` ends the loop.
- `continue;` jumps to next iteration.
- In `foreach` loop, you cannot modify the list that you are iterating on. Instead you have to use a `for` loop for that purpose.

## Random class

```c#
var random = new Random();

Console.WriteLine(random.Next());      //returns a random whole number
Console.WriteLine(random.Next(10));    //less than maxValue
Console.WriteLine(random.Next(10, 20));//between minValue and maxValue
Console.WriteLine(random.NextDouble());//between 0 and 1
```

### Generating random character:

```c#
var random = new Random();

Console.WriteLine((char)random.Next(97, 122));
```

### Password generation example:

```c#
var buffer = new char[10];
var random = new Random();

for (int i = 0; i < 10; i++)
{
    buffer[i] = (char)('a' + random.Next(0, 26)); //when a char concatenate with an int, the result is int
}

var password = new string(buffer);
```

## System.Collections.Generic

### List\<T\>

- In an array, the size is fixed, but by using this type, we can have flexibility.
- Some props and methods:

```c#
var numbers = new List<int>();
numbers.Add(1);                              //Adds to the end of list
numbers.AddRange(new List<int> { 2, 3, 1 }); //Adds an IEnumerable to the end of list
Console.WriteLine(numbers.IndexOf(3));       //2
Console.WriteLine(numbers.LastIndexOf(1));   //3
numbers.Remove(1);                           //Removes the first 1
Console.WriteLine(numbers[0]);               //2: accessing by index
numbers.RemoveAt(numbers.Count - 1);         //Removes the member in a specific index (in this case, the last one)
numbers.Clear();                             //Removes all members

Console.WriteLine(numbers.Count);            //0: a prop
```

- If a method receives a list as an argument, it should clone it, otherwise it will modify the original list:

```c#
var cloned = new List<int>(list);
```

- List\<T\> has a `non-generic` type (ArrayList) defined in `System.Collections`:

```c#
var list = new ArrayList();
list.Add(1);
list.Add("Ben");
list.Add(new Object());

Console.WriteLine(list[1]);//Ben
```

It is not `type-safe` and it has `performance penalty`. Because it receives `object` and wraps everything that it is given into an object (`Boxing`).

### Dictionary<TKey,TValue>

```c#
var dictionary = new Dictionary<string, int>();
dictionary.Add("age", 25);
dictionary["height"] = 185;
Console.WriteLine(dictionary["age"]);                  //25
Console.WriteLine(dictionary["height"]);               //185
dictionary.Remove("age");
Console.WriteLine(dictionary.GetValueOrDefault("age"));//0
Console.WriteLine(dictionary["age"]);                  //KeyNotFoundException

var dict = new Dictionary<string, string>()
{
    ["book"] = "Some sort of written material",
    ["cd"] = "Compact disk"
};
```

### Queue\<T\> and Stack\<T\>

- `Queue<T>` Represents a first-in, first-out (FIFO) collection of objects.
- `Stack<T>` Represents a variable size last-in-first-out (LIFO) collection of instances of the same specified type.

### Collection\<T\>

- **Important:** Actually it is defined in `System.Collections.ObjectModel` not in `System.Collections.Generic`.
- It is like `List<T>`, but without having access to members by `index`.

### Interfaces

- IEnumerable: is read-only; anything that can be enumerated (strings, arrays, lists, dictionaries, ...).
  - **doesn't** have `Add` or `Remove` methods
  - **doesn't** have `Count` prop
  - the members **can't** be accessed by `index`
- ICollection: an IEnumerable with extra:
  - **has** `Add` or `Remove` methods
  - **has** `Count` prop
  - the members **can't** be accessed by `index`
- IList
  - **has** `Add` or `Remove` methods
  - **has** `Count` prop
  - the members **can** be accessed by `index`

## DateTime and TimeSpan

- Both are `struct` and both are `immutable`. So, cannot be changed and always a new value is returned by calling their methods.

```c#
var newYearDay = new DateTime(2020, 1, 1);    //1/1/2020 12:00:00 AM
var today = DateTime.Today;                   //4/12/2020 12:00:00 AM
var yesterday = today.AddDays(-1);            //4/11/2020 12:00:00 AM
// AddSeconds, AddMinutes, AddHours, AddMonths, AddYears

var now = DateTime.Now;                       //4/12/2020 3:10:20 PM

Console.WriteLine(now.ToLongDateString());    //Sunday, April 12, 2020
Console.WriteLine(now.ToShortDateString());   //4/12/2020
Console.WriteLine(now.ToString("yyyy-MM-dd"));//2020-04-12

Console.WriteLine(now.ToLongTimeString());    //3:15:32 PM
Console.WriteLine(now.ToShortTimeString());   //3:15 PM
```

```c#
var oneAndHalfHoursDuration = new TimeSpan(1, 30, 0);      //01:30:00
var oneHourDuration = TimeSpan.FromHours(1);               //01:00:00
var duration = DateTime.Now - DateTime.Now.AddHours(2);    //-02:00:00.00

Console.WriteLine(duration.Minutes);                       //0
Console.WriteLine(duration.TotalMinutes);                  //-120.0
Console.WriteLine(duration.Add(TimeSpan.FromDays(1)));     //21:59:59.9946130
Console.WriteLine(duration.Subtract(TimeSpan.FromDays(1)));//-1.02:00:00.00

Console.WriteLine(duration.ToString());                    //-02:00:00.00
Console.WriteLine(TimeSpan.Parse("01:02:03"));             //01:02:03
```

## Guid

- Is a struct defined in `System` and represents a globally unique identifier (GUID).

```c#
var guid1 = new Guid();                  //Initialize an empty Guid
Console.WriteLine(guid1);                //00000000-0000-0000-0000-000000000000
Console.WriteLine(guid1 == Guid.Empty);  //True

var guid2 = new Guid("abcdef00-ffff-0000-aaaa-abcdefabcdef");  //Fine.
var guid3 = new Guid("a");                                     //Throws FormatException

var guid4 = Guid.NewGuid();                                    //Initializes a new instance of the Guid structure.
Console.WriteLine(guid4);                                      //998f999e-1136-4f74-847a-99290efde9db
var guid5 = Guid.Parse("abcdef00-ffff-0000-aaaa-abcdefabcdef");//Fine.
```

## StringBuilder

It is defined in `System.Text`.

```c#
var builder = new StringBuilder(); //You can pass an initial string

builder.Append('-', 10)
    .AppendLine()
    .Append("Hello")

    .Replace('-', '+')
    .Remove(0, 5)                   //params: startIndex, length
    .Insert(0, new string('*', 5)); //params: index, value

System.Console.WriteLine(builder);
// *****+++++
// Hello
```

## File, FileStreams, Directory, and Path

- Defined in `System.IO`.

### File and FileInfo

- `File` has static methods, whereas `FileInfo` has instance methods.
- When the number of operations is large, it is better to use `FileInfo`, because it checks the permissions once.

```c#
File.Copy(@"d:\sample.txt", @"d:\sample-Copy.txt", true);  //3rd arg is overwrite?
File.Delete(@"d:\sample.txt");
Console.WriteLine(File.Exists(@"d:\sample.txt"));          //False
Console.WriteLine(File.Exists(@"d:\sample-Copy.txt"));     //True
Console.WriteLine(File.ReadAllText(@"d:\sample-Copy.txt"));//"asd"

var fileInfo = new FileInfo(@"d:\sample-Copy.txt");
fileInfo.CopyTo(@"d:\sample.txt", true);                   //2nd arg is overwrite?
fileInfo.Delete();
Console.WriteLine(fileInfo.Exists);                        //False. Here, it is a prop not a method
```

### StreamWriter and StreamReader

- Because file resources are `not` garbage-collected by the `CLR`, we have to `Dispose` them by running Dispose() method on them after we are done (to avoid memory leaks).
- Because reading or writing to a file can throw an exception, we have to enclose the code in a `try-catch-finally` block and put the disposing in the `finally` block.
- We often use `using(){}` blocks, simplify the process of working work with files. So the object will be disposed automatically.

```c#
using (var streamWriter = new StreamWriter(@"d:\sample.txt", append: true))
{
    streamWriter.WriteLine("Hi there!");
}

using (var streamReader = new StreamReader(@"d:\sample.txt"))
{
    var content = streamReader.ReadToEnd();
    Console.WriteLine(content);
}
```

- There are no differences between `streamReader.ReadToEnd()` and `File.ReadAllText(@"d:\sample-Copy.txt")`.
- The difference is when we are using the `ReadLine()` method on an `streamReader object` for large files as it is loading a `chunk` of file into memory rather than the whole file.

### Directory and DirectoryInfo

- `Directory` has static methods, whereas `DirectoryInfo` has instance methods.
- When the number of operations is large, it is better to use `DirectoryInfo`, because it checks the permissions once.

```c#
Directory.CreateDirectory(@"d:\folder1");
Directory.GetFiles(@"d:\folder2", "*.*", SearchOption.AllDirectories); //returns string[]
var dirs = Directory.GetDirectories(@"d:\folder2", "*.*", SearchOption.AllDirectories);
foreach (var dir in dirs)
{
    Console.WriteLine(dir);
}
// d:\folder2\subfolder1
// d:\folder2\subfolder2
Console.WriteLine(Directory.Exists((@"d:\folder1"))); //True

var dirInfo = new DirectoryInfo(@"d:\folder2");
dirInfo.GetFiles();
dirInfo.GetDirectories("*.*", SearchOption.AllDirectories);
```

### Path

```c#
var path = @"d:\sample.txt";
Console.WriteLine(Path.GetExtension(path));               //.txt
Console.WriteLine(Path.GetFileName(path));                //sample.txt
Console.WriteLine(Path.GetFileNameWithoutExtension(path));//sample
Console.WriteLine(Path.GetDirectoryName(path));           //d:\
```

## JSON objects

- `Serialize` means converting data from an object into savable format (JSON).
- `Deserialize` means putting data on an object from previously-saved data (JSON).
- A `JSON` or `JavaScript Object Notation` is a format of storing or transferring data:

```json
{
  "name": "Ben",
  "age": 25,
  "isCool": true,
  "passportNo": null,
  "languages": ["English", "Persian"],
  "children": [
    {
      "name": "Bidel"
    },
    {
      "name": "Delara"
    }
  ],
  "spouse": {
    "name": "Nas"
  }
}
```

- `JsonSerializer` is defined in `System.Text.Json`.
- It has two methods `Serialize` and `Deserialize<T>` to work with JSONs.

```c#
public class Product
{
    public string Name { get; set; }
    public DateTime ExpiryDate { get; set; }
    public decimal Price { get; set; }
    public string[] Sizes { get; set; }

}

class Program
{
    static void Main(string[] args)
    {
        Product product = new Product();

        product.Name = "Apple";
        product.ExpiryDate = new DateTime(2008, 12, 28);
        product.Price = 3.99M;
        product.Sizes = new string[] { "Small", "Medium", "Large" };

        string output = JsonSerializer.Serialize(product);
        File.WriteAllText(@"d:\sample.json", output);
        Console.WriteLine(output);
        //{"Name":"Apple","ExpiryDate":"2008-12-28T00:00:00","Price":3.99,"Sizes":["Small","Medium","Large"]}

        var deserializedProduct = JsonSerializer.Deserialize<Product>(output);
        Console.WriteLine(deserializedProduct.Price); //3.99

        deserializedProduct = JsonSerializer.Deserialize<Product>("{\"Name\":\"Apple\"}");
        Console.WriteLine(deserializedProduct.Name);  //Apple
        Console.WriteLine(deserializedProduct.Price); //0

        deserializedProduct = JsonSerializer.Deserialize<Product>("{\"Id\":1}");
        Console.WriteLine(deserializedProduct.Name); //null

        deserializedProduct = JsonSerializer.Deserialize<Product>("Gibberish"); //Throws JsonReaderException
    }
}
```

- In the old days, we used `Newtonsoft` which you have to install it or come with `.Net Core` (?):
  - JsonConvert.SerializeObject(product) -> returns a string
  - JsonConvert.DeserializeObject\<Product\>(str) -> returns a Product

## Defensive Programming

In the beginning of a method, we should examine the edge-cases and `throw` appropriate `exceptions`. This way, we don't take our program into a `bad state`.

```c#
throw new WhateverException("Some useful message");
```

# Object Oriented Programming

## Constructor Method

- Is a method that is called whenever a new instance of that class is created and puts object in an initial state.
- It must have the same name as the class.
- It has no return.
- We don't need ctors always. If we omit them, the value of each field or property will be the default value of its type. In this case, the `CLR` will generate a `default parameterless` ctor. So technically we always have a ctor even if we don't see it/them.
- It is better to use ctors **only** when:
  1. `Dependency Injection`: when `class A` has a property of the type `class B` (or, of the interface that `class B` also implements), we give an instance of `class B` (which is a concrete implementation in case of using interface) in the ctor of `class A` to initialize that field.
  2. `IEnumerable`: when the class has a property of this type, to avoid the notorious `NullReferenceException`, we initialize the prop in the `ctor`. Technically, we could do this `in-line`, but it is ugly.
- For object initialization, use `object initialization syntax` and not `constructor`!

```c#
public class Customer
{
    public string Name;

    public Customer(string name)
    {
        Name = name;
    }
}

class Program
{
    static void Main(string[] args)
    {
        var customer = new Customer("Ben");
        Console.WriteLine(customer.Name);
    }
}
```

## Constructor Overloading

In any method overloading, the `type` and the `order` of the parameters are determining the `signature` of each method.

```c#
public class Customer
{
    public int Id;
    public string Name;
    public List<Order> Orders; //important: initialize this field!

    public Customer()
    {
        Orders = new List<Order>();
    }

    public Customer(int id) : this()
    {
        Id = id;
    }

    public Customer(int id, string name) : this(id)
    {
        Name = name;
    }
}
class Program
{
    static void Main(string[] args)
    {
        var customer = new Customer(1, "Ben");
        Console.WriteLine(customer.Name);
    }
}
```

- The reality is that it is better to create a `ctor` when is needed; so only the first `ctor` is needed and the next two just made the code ugly and unmaintainable.
- So we should minimize the usage of constructors and instead use the `object initialization syntax`. Otherwise, we have to define ctors with different signatures.

```c#
var person = new Person {FirstName = "Ben", Age = 30};
```

## Method Overloading

```c#
public int X;
public int Y;

public void Move(int x, int y)
{
    X = x;
    Y = y;
}

public void Move(Point newLocation)
{
    if (newLocation == null) //Defensive programming
        throw new ArgumentNullException("newLocation cannot be null");

    Move(newLocation.X, newLocation.Y);
}
```

## Methods Modifiers

### params Modifier

By using the params keyword, you can specify a method parameter that takes a variable number of arguments. The parameter type must be a single-dimensional array.

```c#
public class Calculator
{
    public int Add(params int[] numbers)
    {
        var sum = 0;
        foreach (var number in numbers)
        {
            sum += number;
        }

        return sum;
    }
}

class Program
{
    static void Main(string[] args)
    {
        var calc = new Calculator();
        Console.WriteLine(calc.Add(1, 2, 3, 4, 5, 6, 7)); //28
        Console.WriteLine(calc.Add(new int[] { 1, 2, 3, 4, 5, 6, 7 }));//28
    }
}
```

### ref Modifier

If you pass a `value type` into a method, the method can manipulate it like `reference type`.

```c#
static void Main(string[] args)
{
    int a = 1;
    int b = 1;
    Weird(ref a, b);      //We have to specify `ref` when using
    Console.WriteLine(a); // 0 (!)
    Console.WriteLine(b); // 1
}

static void Weird(ref int a, int b)
{
    a = 0;
    b = 0;
}
```

### out Modifier

```c#
static void Main(string[] args)
{
    int sum;
    int mul = Weird(1, 2, out sum);
    Console.WriteLine(sum); // 3 (!)
}

static int Weird(int a, int b, out int sum)
{
    sum = a + b;
    return a * b;
}
```

In both `ref` and `out` modifiers, we have to use a variable which is already defined when calling the method. In `out` modifier, you have to assign something to the `out` parameter. A real-world example:

```c#
int number;
var result = int.TryParse("xyz", out number); // result type is bool
if (result)
{
    Console.WriteLine(number);
}
else
{
    Console.WriteLine("Failed");
}
```

instead of:

```c#
try
{
    var result = int.Parse("xyz");
    Console.WriteLine(result);
}
catch (Exception)
{
    Console.WriteLine("Failed");
}
```

## Encapsulation (Hiding info.)

### Field or Property Set Accessors

1. `readonly` fields:
   - from outside: cannot be set.
   - from inside: only can be set **once**.
2. props with `private set`:
   - from outside: cannot be set.
   - from inside: can be set everywhere.
3. props **without** `set`:
   - from outside: cannot be set.
   - from inside: can be set only in the ctor.

### Class or Member (Field, Property, or Method) Get Accessors (Access Modifiers)

1. `public` (for both members and classes):
   - from outside: can be read.
   - from derived classes: can be read.
   - from inside: can be read.
1. `protected` (only for members):
   - from outside: cannot be read.
   - from derived classes: can be read.
   - from inside: can be read.
   - In general, it is better, not to define a member `protected` and instead make it `private`; because if we change or remove a protected member but the children have used it, we will have a problem.
1. `private` (only for members):
   - from outside: cannot be read.
   - from derived classes: cannot be read.
   - from inside: can be read.
1. `internal` (for both members and classes):
   - Internal types or members are accessible only within files in the `same assembly`.
1. `protected internal` (only for members):
   - A protected internal member is accessible from the current assembly `or` from types that are derived from the containing class.

In **Java**, we make fields private and provide public `getters` and `setters` in order to have some logic in accessing fields. But in `C#`, we use **`public props`** and **`private fields`**:

- public props (PascalCase): to have logic in accessing fields.
- The props seem to be fields (they are used like fields) but actually they are setters and getters methods.

```c#
public class Person
{
    public DateTime Birthday { get; private set; }

    public Person(DateTime birthday)
    {
        Birthday = birthday;
    }

    public int Age
    {
        get
        {
            return DateTime.Now.Year - Birthday.Year;
        }
    }

}
class Program
{
    static void Main(string[] args)
    {
        var person = new Person(new DateTime(1985, 1, 1));
        System.Console.WriteLine(person.Age); //35
    }
}
```

- _Convention_: First `Auto-implemented props`, then `ctors`, and finally `calculated props`.
- If a prop has just one `get` (like the above example), we can use `expression bodied prop`:

```c#
public int Age => DateTime.Now.Year - Birthday.Year;
```

- If there is no logic in the prop, we use `Auto-implemented properties`:

```c#
public int MyProperty { get; set; }
```

- private fields (\_camelCase): for things that are `implementations details` or for implementing `dependency injection`.

### Indexer Property

Indexer is a **property**:

```c#
public class HttpCookie
{
    private readonly Dictionary<string, string> _dictionary;

    public HttpCookie()
    {
        _dictionary = new Dictionary<string, string>();
    }

    public string this[string key]
    {
        get { return _dictionary[key]; }
        set { _dictionary[key] = value; } //value is the RHS of the property assignment
    }
}
class Program
{
    static void Main(string[] args)
    {
        var httpCookie = new HttpCookie();
        httpCookie["Authorization"] = "Bearer xyx...";
        Console.WriteLine(httpCookie["Authorization"]);
    }
}
```

## Inheritance

- We have `code-reuse` and we have access to `methods` and `props` of the parent class, in an each instance of the child class.
- Each class in `C#` can be derived only from `one` class. We do not have `multiple inheritance` in `C#`.
- `Object` class is the parent of all classes.

```c#
public class Parent
{
    //...
}
public class Child : Parent
{
    //...
}
```

- When instantiating a Child class, always the ctor of the Parent will be executed first (remember that if we don't have one, a `default parameterless` ctor will be created by the `CLR`).
- If the Parent class has a ctor that receives a parameter (and at the same time the Parent does not have any parameterless ctor), we `must` call the Parent's ctor in the child ctor:

```c#
public class Parent
{
    private readonly Options _options;
    public Parent(Options options)
    {
        _options = options;
    }
}

public class Child : Parent
{
    public Child(Options options) : base(options)
    {
        //...
    }
}
```

### Up-casting

- The conversion of a child class to its parent. The interface will be reduced.
- It is done implicitly and we don't have to do anything.
- It is usually done in the method parameters.

```c#
static void Main(string[] args)
{
    var child = new Child();
    DoSomething(child);
}
static void DoSomething(Parent parent)
{
    //...
}
```

### Down-casting

- The conversion of a parent class to the child class. The interface will be increased to have access to more members.
- It must be done explicitly and it may throw `InvalidCastException`.
- For down-casting, the object must be of the `Child` type `originally`. You cannot forcefully downcast a Parent object into a child.

```c#
static void Main(string[] args)
{
    var parent = new Parent();
    var child = new Child();
    DoSomething(child); //It is fine.
    DoSomething(parent);//It throws exception.
}
static void DoSomething(Parent parent)
{
    var child = (Child)parent; //Down-casting
}
```

### `is` and `as` Keywords

Because down-casting might throw, it is better to use `as` keyword. In this case, if it couldn't convert the type, the object will be null.

```c#
var parent = new Parent();
var child = new Child();
var childInParentsClothes = (Parent)child;

Console.WriteLine(childInParentsClothes is Child);//True
Console.WriteLine(parent is Child);               //False

var childInChildsClothes = childInParentsClothes as Child;
var parentInChildsClothes = parent as Child;

Console.WriteLine(childInChildsClothes != null);  //True
Console.WriteLine(parentInChildsClothes != null); //False
```

### Method Overriding

- It is changing the implementation of an inherited method.
- In the parent's method that we want to be overridable, we use `virtual` keyword.
- In the child's method that we want to override, we use `override` keyword.
- With `method overriding`, we reach **`Polymorphism`**.

```c#
public class Parent
{
    public virtual void DoSomething()
    {
        Console.WriteLine("Original implementation");
    }
}

public class Child1 : Parent
{
    public override void DoSomething()
    {
        Console.WriteLine("Child1's implementation");
    }
}
public class Child2 : Parent
{
    public override void DoSomething()
    {
        Console.WriteLine("Child2's implementation");
    }
}
class Program
{
    static void Main(string[] args)
    {
        Work(new List<Parent> { new Child1(), new Child2() });
        //Child1's implementation
        //Child2's implementation
    }
    static void Work(List<Parent> parents)
    {
        foreach (var parent in parents)
        {
            parent.DoSomething();
        }
    }
}
```

### `sealed` Classes

- No class can be derived from a `sealed` class.
- Sealed classes are a little bit faster. Not very common though.
- `string` class is sealed and we must `extend` it.

```c#
public sealed class Infertile
{
}
```

### `sealed override` Methods

```c#
public class Parent
{
    public virtual void DoSomething()
    {
        Console.WriteLine("Original implementation");
    }
}

public class Child : Parent
{
    public sealed override void DoSomething()
    {
        Console.WriteLine("Child's implementation");
    }
}

public class GrandChild : Child
{
    public override void DoSomething() //Compile Error: cannot override inherited member
    {
        Console.WriteLine("GrandChild's implementation");
    }
}
```

### Abstract Methods and Classes

- In the example in the `method overriding` section, we could also use `abstract` keyword to reach `polymorphic behavior`.
- abstract methods, don't have any implementation.
- If a class has an abstract method, it should mark as abstract as well. Of course an abstract class can have non-abstract methods too for common behavior.
- Abstract classes cannot be instantiated.
- All the derived classes from an abstract class `must` implement those abstract methods. So it is better than creating an empty virtual method.

```c#
public abstract class Parent
{
    public abstract void DoSomething(); //No implementation
}

public class Child1 : Parent
{
    public override void DoSomething()
    {
        Console.WriteLine("Child1's implementation");
    }
}
```

## Objects Relationships

1. Inheritance: is-a relationship: class A extends class B.
1. Association: has-a relationship: class A has a field/prop of the type of class B:
   - Composition: has-one (1-1->1-1 ): Car and engine
   - Aggregation: has-any-number-of (0-1->0-\*): Car and passengers
1. Dependency: uses-a relationship:

   - class A accepts a parameter of type class B in one of its methods or
   - class A instantiates an object of type class B as a local variable in one of its methods
   - class A returns an object of type class B in one of its methods

     ![](/md/relationships.jpg)

- **Important**: from top to bottom, the relationship is getting weaker.
- Inheritance > Composition > Aggregation > Dependency
- In both cases of `Inheritance` and `composition` we did `code-reuse`.
- In Inheritance we have access to all the methods of the parent in an instance of child, maybe we don't want that and we want to expose only some specific parent's methods or their modifications, in this case, we use composition with a private field.
- In Inheritance we have `tight-coupling` but in composition we have a weaker relationship (**If** we use `dependency injection` we have `loose-coupling`. It will be discussed later.) so prefer to use `composition` over `Inheritance`:

  > A `Person` has an `Animal` and a `Walkable`.

  > A `Dog` has an `Animal` and a `Walkable`.

  > A `Fish` has an `Animal` and a `Swimmable`.

  > A `Duck` has an `Animal`, a `Walkable`, and a `Swimmable`.

- Inheritance can be mis-used and generate a fragile hierarchy.

## Interfaces

```css
interface ISampleInterface
{
    int SampleProp { get; set; };
    void SampleMethod();
}
```

- It is a contract for the types that implements the `interface`.
- The name should start with `I` followed by a Capital letter.
- The props and methods declared in the interface don't have access modifiers. Because they are public always.
- In the interface we can only define `props` and `methods` and **not** `fields`.
- The methods declared in the interfaces, do not have a body.
- When you implement an interface, you have to write **all** the props and methods that the interface declares. They also must be **public**.
- In `virtual` and `abstract` methods, we used `override` keyword but here we just write the method normally.
- We can implement multiple interfaces but interfaces are **not** for `multiple inheritance`! Because we **don't** have code re-use with interfaces.
- An interface can also `inherit` from another interfaces.
- Whenever, we have an `abstract` class without any `concrete` methods, there is no `code-reuse` and we should change the `abstract` class to an `interface`.
- It is true that we don't have code-reuse with interfaces themselves, BUT they enable us to have more generic methods which reduces duplications.

## Dependency Injection

- We are `using` a `class` but we `inject` an `instance`.
- If we instantiate a `new` instance of class B inside class A in a composition relationship, we have tight coupling, which is bad (but still better than Inheritance).
- In composition, it is more `loosely-coupled` if we pass an instance of the other class instead of `newing` (which is called `dependency injection`):

```c#
public class Installer
{
    private Logger _logger;

    public Installer(Logger logger)
    {
        _logger = logger;
    }
}
```

- It is even better that we talk to an interface in class A and pass a concrete implementation of class B in the ctor:

```c#
public class Installer
{
    private ILogger _logger;

    public Installer(ILogger logger)
    {
        _logger = logger;
    }
}
```

- In summary, when `class A` has a property of the type of the interface that `class B` also implements, we give an instance of `class B` as a concrete implementation in the ctor of `class A` to initialize that field.
  - Apart from injecting the object in the `ctor` which is done **often**, we also could inject it in a `method` or in a `prop`.
- This technique is often used in `unit test`, when the `fake class B` also implements the interface.

For comparing the `flexibility`:

> Inheritance < Composition with newing < Dependency Injection with concrete classes < Dependency Injection with interfaces

- **Note**: We restrict the use of `dependency injection` for the classes that deals with `external resources`. Do not extract an interface for everything!

### Dependency Injection Frameworks

- with `DI frameworks` a `new instance` of the injected class (or a concrete implementation of an interface) will be passed to the constructor (_usually_) of the classes that receive that injected class (or an object of that interface type) at runtime, if we don't pass a specific implementation ourselves. So we have to register an initial state for that injected object (or the interface and the concrete implementation counterpart) in the `DI` framework. In summary, with `DI Frameworks`, we write code like this:

```c#
public Handler(DataContext context, IPhotoStorage photoStorage)
{
    _context = context;
    _photoStorage = photoStorage;
}
```

instead of:

```c#
public Handler(DataContext context = null, IPhotoStorage photoStorage = null)
{
    _context = context ?? new DataContext(//some options to pass the connection info for connecting to Db);
    _photoStorage = photoStorage ?? new PhotoStorage();
}
```

# Advanced

## Generics

By `generics`, we don't need to different `types` of list and we can use a `generic` class.

```c#
public class GenericList<T>
{
    public void Add(T value)
    {
        //...
    }

    public T this[int index]
    {
        get { throw new NotImplementedException() }
    }
}

class Program
{
    static void Main(string[] args)
    {
        var numbers = new GenericList<int>();
        numbers.Add(10);

        var books = new GenericList<Book>();
        books.Add(new Book());
    }
}
```

There are `5` types of `constraints` in generic classes:

1. `where T : ISomeInterface`, where that type should implement a specific interface.

```c#
public class Utilities<T> where T : IComparable
{
    public T Max(T a, T b)
    {
        return a.CompareTo(b) > 0 ? a : b;
    }
}

class Program
{
    static void Main(string[] args)
    {
        var utilities = new Utilities<int>();
        Console.WriteLine(utilities.Max(1, 3));
    }
}
```

2. `where T : Parent`

```c#
public class DiscountCalculator<TProduct> where TProduct : Product
{
    public float CalculateDiscount(TProduct product)
    {
        return product.Price*0.1f;
    }
}

public class Product
{
    public float Price { get; set; }
}
```

3. `where T : struct`, means that the T must be a `value type`.

```c#
public class Nullable<T> where T : struct
{
    private Object _value;

    public Nullable()
    {
    }

    public Nullable(T value)
    {
        _value = value;
    }

    public T Value
    {
        get
        {
            return _value != null ? (T)_value : throw new Exception("InvalidOperationException");
        }
        set
        {
            _value = value;
        }
    }
    public bool HasValue { get { return _value != null; } }

    public T GetValueOrDefault()
    {
        return HasValue ? (T)_value : default(T);
    }
}

class Program
{
    static void Main(string[] args)
    {
        var number1 = new Nullable<int>(5);
        Console.WriteLine(number1.HasValue);           //True
        Console.WriteLine(number1.GetValueOrDefault());//5
        Console.WriteLine(number1.Value);              //5

        var number2 = new Nullable<int>();
        Console.WriteLine(number2.HasValue);           //False
        number2.Value = 1;
        Console.WriteLine(number2.HasValue);           //True
        Console.WriteLine(number2.GetValueOrDefault());//1
        Console.WriteLine(number2.Value);              //1

        var number3 = new Nullable<int>();
        Console.WriteLine(number3.HasValue);           //False
        Console.WriteLine(number3.GetValueOrDefault());//0
        Console.WriteLine(number3.Value);              //Throws Error
    }
}
```

Note that in `.Net`, the `Nullable<>` type is defined as a `struct` and not a class.

4. `where T : class`, means that the T must be a `reference type`.

5. `where T : new()`, means that the T must have a default ctor.

```c#
public class Utilities<T> where T : new()
{
    public void DoSomething()
    {
        var obj = new T(); // If you want to create an instance of the variable type, it has to have a default ctor.
    }
}
```

**important**: If you want to combine the constraints above, you can list them with a `,` separator.

## Delegates (Action and Func)

- When we want to pass a **`function as a parameter`** (an object) to another method, we use `delegates`.
- There are two classes (types) of delegates in the `.Net` framework: `Action` and `Func` which are **`Generic Classes`**
  - `Func<T1, T2, ..., Tn>` is used when there is a return (`Tn` is the return type) from the method that we want to pass.
  - `Action<T1, T2, ..., Tn>` is used when there is no return (`void`) from the method that we want to pass and all T1, T2, ... are the parameters of the method that we want to pass.
- The **`function as a parameter`** (an object) that we want to pass can be

  - an `instance method`
  - a `static method`
  - a `Lambda expression`
  - a `new instance` of that delegate class (Func<> or Action<>). So for example, a function is an instance of Func<> class. With this, we can reach `Functional programming paradigm`

    as long as it has the signature of the delegate class.

- So this object `knows` how to call a method or an anonymous function.
- If you want to check that if a delegate is not null before invoking, you can use `Action sayHi = () => Console.WriteLine("Hi there"); sayHi?.Invoke();` instead of `Action sayHi = () => Console.WriteLine("Hi there"); if (sayHi != null) { sayHi(); }`.

```c#
public class Photo
{
    public string Path { get; private set; }

    public void Save()
    {
        Console.WriteLine("Saving photo...");
    }

    public static Photo Load(string path)
    {
        return new Photo { Path = path };
    }
}

public class PhotoProcessor
{
    private readonly Photo _photo;
    private readonly List<Action<Photo>> _filters;

    public PhotoProcessor(string path)
    {
        _photo = Photo.Load(path);
        _filters = new List<Action<Photo>>();
    }

    public void Process()
    {
        foreach (var filter in _filters)
        {
            filter(_photo);
        }

        _photo.Save();
    }

    public void AddFilter(Action<Photo> filter)
    {
        _filters.Add(filter);
    }
}

public class DefaultFilters
{
    public void ReSize(Photo photo)
    {
        Console.WriteLine("Resizing...");
    }

    public static void AdjustContrast(Photo photo)
    {
        Console.WriteLine("Adjusting contrast...");
    }
}

class Program
{
    static void Main(string[] args)
    {
        var photoProcessor = new PhotoProcessor(@"d:\sample.jpg");

        var defaultFilters = new DefaultFilters();
        photoProcessor.AddFilter(defaultFilters.ReSize);//instance method

        photoProcessor.AddFilter(DefaultFilters.AdjustContrast);//static method

        photoProcessor.AddFilter(photo => Console.WriteLine("Adjusting exposure..."));//Lambda expression

        Action<Photo> redEyeFilter = photo => Console.WriteLine("Removing red eyes...");
        photoProcessor.AddFilter(redEyeFilter);//new instance of the delegate class

        var cropFilter = new Action<Photo>(photo => Console.WriteLine("Cropping..."));
        photoProcessor.AddFilter(cropFilter);//new instance of the delegate class

        photoProcessor.Process();
        //Resizing...
        //Adjusting contrast...
        //Adjusting exposure...
        //Removing red eyes...
        //Cropping...
        //Saving photo...
    }
}
```

We could use `interfaces` to solve the problem above. We use delegates when the caller does not need other properties or methods of the object that the delegate points to one of its methods.

## Lambda Expressions

- Is an anonymous method.
- They are usually associated with delegates. So, instead of assigning a method to a delegate, we assign a `lambda expression` to `Func` delegates or `lambda statement` to `Action` delegates:

```c#
Func<int> whatYear = () => DateTime.Now.Year; //Needs ()
Func<int, int> square = n => n * n;           //Doesn't need ()
Func<int, int, int> add = (a, b) => a + b;    //Needs ()

Console.WriteLine(whatYear());   //2020
Console.WriteLine(square(5));    //25
Console.WriteLine(add(5, 5));    //10

Action<string> greet = name => Console.WriteLine($"Hi {name}");
greet("Ben");                    // Hi Ben
```

- `Predicate` is a delegate of type `Func<CollectionType, bool>`.

```c#
public class Book
{
    public string Title { get; set; }
    public float Price { get; set; }
}

public class BookRepository
{
    public IEnumerable<Book> GetBooks()
    {
        return new List<Book>{
            new Book{Title = "Title1", Price = 12.99f},
            new Book{Title = "Title2", Price = 8.99f},
            new Book{Title = "Title3", Price = 15.99f},
            new Book{Title = "Title4", Price = 9.99f},
            new Book{Title = "Title5", Price = 2.99f},
        };
    }
}

class Program
{
    static void Main(string[] args)
    {
        var books = new BookRepository().GetBooks();
        var cheapBooks = books.Where(b => b.Price < 10);

        foreach (var book in cheapBooks)
        {
            Console.WriteLine(book.Title);
        }
        //Title2
        //Title4
        //Title5
    }
}
```

## Events

- First, define an `EventHandler` delegate (it is defined in `.Net` and returns `void`) in the `publisher` (`emitter`) class.
  - If we don't want to pass data to `EventHandler`, we use `non-generic` type: `EventHandler`
  - If we want to pass data to `EventHandler`, we use `generic` type: `EventHandler<Video>`

Passing data example:

```c#
public class Video
{
    public string Title { get; set; }
}

public class VideoEncoder
{
    public event EventHandler<Video> VideoEncoded; //Publisher class
    public void Encode(Video video)
    {
        Console.WriteLine("Encoding video...");

        Thread.Sleep(1000); //It stops the running thread for 1 second

        OnVideoEncoded(video);
    }

    protected virtual void OnVideoEncoded(Video video) //It is convention to mark this method as protected virtual void
    {
        VideoEncoded?.Invoke(this, video); //This ? is for checking not null: null propagation
    }
}

public class MailService //Subscriber class
{
    public void OnVideoEncoded(object source, Video video)
    {
        Console.WriteLine($"Sending email for video {video.Title}");
    }
}

class Program
{
    static void Main(string[] args)
    {
        var video = new Video();
        var videoEncoder = new VideoEncoder();
        var mailService = new MailService();

        videoEncoder.VideoEncoded += mailService.OnVideoEncoded; //Registering.
        videoEncoder.VideoEncoded += (source, video) => Console.WriteLine("Encoded."); ; //You can even register a delegate in the form of a lambda expression.

        videoEncoder.Encode(new Video { Title = "Shrek" });
        //Encoding video...
        //Sending email for video Shrek
        //Encoded.
    }
}
```

Without passing data example:

```c#
public class Video
{
    public string Title { get; set; }
}

public class VideoEncoder
{
    public event EventHandler VideoEncoded;
    public void Encode(Video video)
    {
        Console.WriteLine("Encoding video...");

        Thread.Sleep(1000);

        OnVideoEncoded();
    }

    protected virtual void OnVideoEncoded() => VideoEncoded?.Invoke(this, EventArgs.Empty);
}

public class MailService
{
    public void OnVideoEncoded(object source, EventArgs e) => Console.WriteLine($"Sending email...");
}

class Program
{
    static void Main(string[] args)
    {
        var video = new Video();
        var videoEncoder = new VideoEncoder();
        var mailService = new MailService();

        videoEncoder.VideoEncoded += mailService.OnVideoEncoded; //registering

        videoEncoder.Encode(new Video { Title = "Shrek" });
        //Encoding video...
        //Sending email...
    }
}
```

## Extensions Methods

- For example `string` class has been defined `sealed` and we cannot derive from it. So we have to extend it.
- The extension methods are defined as `static` in a `static` class (usually called `OriginalClassExtensions`) but because the first parameter is of type `this OriginalClass`, they are used like instance methods (with a different icon of course).
- It is recommended not to use `extensions methods` much, because if `Microsoft` decides to define a new instance method with the same name, it will take precedent and our method won't run.

```c#
public static class StringExtensions
{
    public static string Shorten(this string str, int numberOfWords)
    {
        var words = str.Split(' ');

        return string.Join(" ", words.Take(numberOfWords)); //Take is defined in Linq
    }
}
class Program
{
    static void Main(string[] args)
    {
        var str = "This is a very very very very very very very very very long text...";
        System.Console.WriteLine(str.Shorten(5)); //This is a very very
    }
}
```

## Linq

- Is defined in `System.Linq`.
- Is short for `Language Integrated Query`.
- It can be used to query data in memory, database, XML, ADO, ...
- The methods in the example below is called `Linq extensions methods`. We have another form of query which is called `Linq Query Operators` which is similar to `SQL` (I don't like it).

```c#
var books = new BookRepository().GetBooks(); //Check the example in Lambda expressions section

var cheapBooks = books
    .Where(b => b.Price < 10) //Because each method returns an IEnumerable, we can chain them
    .OrderByDescending(b => b.Price)
    .Select(b => b.Title); //It is like Map in JS

foreach (var book in cheapBooks)
{
    Console.WriteLine(book);
}
//Title4
//Title2
//Title5

var pagedBooks = books.Skip(2).Take(3);

foreach (var book in pagedBooks)
{
    Console.WriteLine(book.Title);
}
//Title3
//Title4
//Title5
```

- The methods above return `IEnumerable<>` which has a very limited interface (no indexer, no Count prop, even no Count() method {the Count() method is actually a Linq extensions method}, no Add, Remove,...). So if you want to work with those features, use `ToList` method on them which returns a `List` or instantiate a new `List`:

```c#
var pagedBooksList = pagedBooks.ToList();
Console.WriteLine(pagedBooksList.Count); //3

//or
var pagedBooksList = new List<Book>(pagedBooks);
Console.WriteLine(pagedBooksList.Count); //3
```

- Methods that return one object:

```c#
var book1 = books.Single(b => b.Title == "Title2"); //It returns one Book. If there is no book or more than one books, it throws InvalidOperationException.
Console.WriteLine(book1.Title); //Title2

var book2 = books.SingleOrDefault(b => b.Title == "Title6"); //It returns one Book. If there is no book, it return the default value (null in this case). If there are more than one books, it throws InvalidOperationException.
Console.WriteLine(book2?.Title); //nothing

var book3 = books.First(); //It returns the first Book. If there is no book, it throws InvalidOperationException.
Console.WriteLine(book3.Title); //Title1

var book4 = books.First(b => b.Title == "Title2"); //It returns the first Book that matches the condition. If there is no book, it throws InvalidOperationException.
Console.WriteLine(book4.Title); //Title2

var book5 = books.FirstOrDefault(b => b.Title == "Title6"); //It returns the first Book that matches the condition. If there is no book, it return the default value (null in this case).
Console.WriteLine(book5?.Title); //nothing

var book6 = books.Last(b => b.Title == "Title2"); //It returns the last Book that matches the condition. If there is no book, it throws InvalidOperationException.
Console.WriteLine(book6.Title); //Title2

var book7 = books.LastOrDefault(b => b.Title == "Title6"); //It returns the last Book that matches the condition. If there is no book, it return the default value (null in this case).
Console.WriteLine(book7?.Title); //nothing
```

- Aggregate methods:

```c#
Console.WriteLine(books.Count());              //5
Console.WriteLine(books.Max(b => b.Title));    //Title5
Console.WriteLine(books.Min(b => b.Price));    //2.99
Console.WriteLine(books.Sum(b => b.Price));    //50.95
Console.WriteLine(books.Average(b => b.Price));//10.19
```

## Nullable

- `Value types` cannot be null.

```c#
DateTime date = null; //Won't compile
Nullable<DateTime> date1 = DateTime.Now; //Nullable is a generic struct in System namespace
DateTime? date2 = null; //Shorter way to define nullables

Console.WriteLine(date1.HasValue);           //True
Console.WriteLine(date1.GetValueOrDefault());//4/13/2020 8:22:50 PM
Console.WriteLine(date1.Value);              //4/13/2020 8:22:50 PM

Console.WriteLine(date2.HasValue);           //False
Console.WriteLine(date2.GetValueOrDefault());//1/1/0001 12:00:00 AM
Console.WriteLine(date2.Value);              //Throws InvalidOperationException:

DateTime date3 = date1.GetValueOrDefault();  //If we omit GetValueOrDefault(), it won't compile, because it doesn't know what to do it date1 is null.
DateTime? date4 = date3;                     //No problem here. A value type can be casted to a nullable type.

DateTime date5 = (date1 != null) ? date1.GetValueOrDefault() : DateTime.Today;
DateTime date6 = date1 ?? DateTime.Today;    //The same as the above. Null Coalescing Operator.
```

## dynamic

- With `dynamic`, `C#` becomes `js`!

```c#
dynamic name = "Ben";
name = 5;
```

- `DLR` sits on top of `CLR`.
- Note that if we invoke a method which is not compatible with the type at runtime, we will get `RuntimeBinderException`.
- With `dynamic` types, we have to write more `unit test`.
- Note that `dynamic` is the opposite of `var`. With `var`, the compiler decides what is the `static` type.
- If there is one `dynamic` in an expression, the whole expression will be evaluated as `dynamic`:

```c#
dynamic a = 10;
int b = 5;
var c = a + b; //c is dynamic
```

- Casting:

```c#
int i = 5;
dynamic d1 = i;
dynamic d2 = "Ben";
long l1 = d1; //It's fine because at the runtime, d1 is int, and int is implicitly convertible to long.
long l2 = d2; //throws RuntimeBinderException
```

## Exception Handling

- If we have multiple `catch` blocks, we should sort if from most specific to the most general, because only the first match is run.

```c#
var a = 0;
var b = 1;
int c;
try
{
    c = b / a;
}
catch (DivideByZeroException ex)
{
    Console.WriteLine("Division by zero: " + ex.Message);
}
catch (ArithmeticException ex)
{
    Console.WriteLine("Bad arithmetic operation: " + ex.Message);
}
catch (Exception ex)
{
    Console.WriteLine("Something went wrong! " + ex.Message);
}
```

- `finally` block is run in both case of `success` or `failure` and is the best place to `dispose` un-managed resources:

```c#
StreamReader streamReader = null;
try
{
    streamReader = new StreamReader(@"d:\sample.txt");
    var content = streamReader.ReadToEnd();
    throw new Exception("Oops");
}
catch (Exception)
{
    Console.WriteLine("Something went wrong!");
}
finally
{
    Console.WriteLine("I will run always!");
    streamReader?.Dispose();
}
```

- By using `using` block, we don't have to dispose the reader object manually:

```c#
try
{
    using (var streamReader = new StreamReader(@"d:\sample.txt"))
    {
        var content = streamReader.ReadToEnd();
        throw new Exception("Oops");
    }
}
catch (Exception)
{
    Console.WriteLine("Something went wrong!");
}
```

### Custom Exceptions

```c#
public class CustomException : Exception
{
    public CustomException(string message, Exception innerException) : base(message, innerException)
    {
    }
}

public class Api
{
    public List<Video> GetVideos()
    {
        try
        {
            Console.WriteLine("Trying to retrieve videos...");
            throw new Exception("Very technical error message");
        }
        catch (Exception ex)
        {
            throw new CustomException("Unable to get videos", ex);
        }
    }
}

public class Video
{
}

class Program
{
    static void Main(string[] args)
    {
        try
        {
            var api = new Api();
            var videos = api.GetVideos();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message); //Unable to get videos
        }
    }
}
```

## Pattern matching switch statement

```c#
switch (ex)
{
    case RestException restException:
        Console.WriteLine(ex, "REST ERROR");
        break;
    case Exception otherException:
        Console.WriteLine(ex, "SERVER ERROR");
        break;
}
```

## async-await

- When dealing with `web`, `database`, `file system`, ... it takes time. So if we `wait` for the operation to complete and `don't release` the thread (`blocking`), no more requests can be handled by our server (In case of API applications). Which is very bad.
- In the past, we made use of `multi-threading`, `callbacks`, ... but now we have `async-await`.

### Usage

- When the controller reaches to an `await`, the thread will be released and so the application can respond to other requests and when the `response` comes back, the rest will be run.
- As a rule of thumb, if a method has an `async` form, use it.
- The convention is when a method is `async`, the name should end with `Async`.
- When using `await` before an execution of a method (before method call and after "=") in a method, we change the return type of the method:
  - If a method returns `void` -> `async Task`.
  - If a method returns `Something` -> `async Task<Something>`.
- `Task` is like `Promise` in `js`.

```c#
static async Task Main(string[] args)
{
    var streamReader = new StreamReader(@"d:\sample.txt");
    var content = await streamReader.ReadLineAsync();
    Console.WriteLine(content);
}
```

- With the technique below, we don't have to use `await` right away:

```c#
var customTask = CustomAsync(); //because an async method returns a task
MessageBox.Show("Waiting for task to be completed...");
var result = await customTask; //we await here, after showing the window
MessageBox.Show(result);

```

- If we have an `async` operation but we want to block the rest of the execution (it is a rare case), until the response comes back (an example is when we want to seed the data on the starting of our server, so we have a line like the following in the `Program.cs` class):

```c#
Seed.SeedData(context, userManager).Wait()
```

If the method is returning `void`, we simply use `.Wait()` at the end. If the method returns `Something`, we append `.Result` at the end. In this case, we `shouldn't` change the method signature to `async Task`.

## SOLID Principles

### Single Responsibility

- The single responsibility principle (SRP) states that every class in a program should have responsibility for just a single piece of that program's functionality.
- If a class has a lot of `private` methods with many execution paths, it is indicating that those private methods actually belong to another class and should be public.

### Open-closed principle

- In software engineering, it means that the entities should be open for extensions but closed for modifications.

```c#
public class Video
{
}

public interface INotificationChannel
{
    void Send();
}

public class SmsNotificationChannel : INotificationChannel
{
    public void Send()
    {
        Console.WriteLine("Sending sms...");
    }
}

public class VideoEncoder
{
    private readonly List<INotificationChannel> _notificationChannels;

    public VideoEncoder()
    {
        _notificationChannels = new List<INotificationChannel>();
    }

    public void Encode(Video video)
    {
        Console.WriteLine("Encoding video...");

        foreach (var channel in _notificationChannels)
        {
            channel.Send();
        }
    }

    public void RegisterNotificationChannel(INotificationChannel channel)
    {
        _notificationChannels.Add(channel);
    }

}

class Program
{
    static void Main(string[] args)
    {
        var video = new Video();
        var smsNotificationChannel = new SmsNotificationChannel();
        var videoEncoder = new VideoEncoder();

        videoEncoder.RegisterNotificationChannel(smsNotificationChannel);
        videoEncoder.Encode(video);
        //Encoding video...
        //Sending sms...
    }
}
```

### Liskov substitution principle

Derived classes must be substitutable for the base class without unexpected behavior.

### Interface segregation principle

No client should be forced to depend on methods it does not use. Many client-specific interfaces are better than one general purpose interface.

### Dependency Inversion Principle

High-level modules should not depend on low-level modules (implementations). Both should depend on abstractions (interfaces).
