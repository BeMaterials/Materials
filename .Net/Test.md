# Setup

```dos
dotnet new nunit -n Playground.UnitTests
dotnet sln add Playground.UnitTests/
cd Playground.UnitTests
dotnet add reference ../Playground/
cd..
dotnet test
```

# Introduction

## Naming Convention

- The name of the test project should be `the name of the main project + .UnitTests`. So for every project in the solution, there is one unit test project.
- We implement the `same folder structure` as the main project.
- The name of test classes should be `the name of counterpart classes + Tests`.
- The name of the test method should be: `TheNameOfMainMethod_Scenario_ExpectedBehavior`.

## NUnit

- All test methods should be `public void`.
- Test classes should be decorated with `[TestFixture]` attribute.
- Test methods should be decorated with `[Test]` attribute.
- Method decorated with `[SetUp]` attribute will run before every test method. Because we want every test to be independent of the other tests,
  - we introduce a private field of the type of the class to be tested in the test class and
  - initialize with a new instance in the `[SetUp]` method.
- Method decorated with `[TearDown]` attribute will run after every test method. For example, when dealing with databases, we clean after ourselves in this method.

## Parts of Each Unit Test

- `Arrange`: we initialize a new object of the class to be tested.
- `Act`: we call the method that we want to test.
- `Assert`: we verify that the result is correct.

## General Rules of Unit Testing

- Test should `not` have `logic`.
- The subject should be `isolated`.
- The test should `not` be `too specific` or `too general`.
- Clean coding increase the testability.
- We test an output of a method (query/command). In case of commands, they change the `state` of an object, database, send a message, ... and we should test that.
- You should not test language features or third-party libraries.
- Unit test should be run often but `integration tests` before committing to the repository.
- When you code first (instead of TDD: Test Driven Development), sometimes change the code to make sure that the tests are `trustworthy`.
- `private` and `protected` methods are `implementation details` and change often; so we don't write unit tests for them.

## First Unit Test

```c#
[TestFixture]
public class CalculatorTests
{
    private Calculator _calculator;

    [SetUp]
    public void Setup()
    {
        _calculator = new Calculator();
    }

    [Test]
    public void Add_WhenCalled_ReturnTheSumOfArguments()
    {
        var result = _calculator.Add(1, 2);

        Assert.That(result, Is.EqualTo(3));
    }


}
```

## Parameterized Test

```c#
[Test]
[TestCase(2, 1, 2)]
[TestCase(1, 2, 2)]
[TestCase(1, 1, 1)]
public void Max_WhenCalled_ReturnTheGreaterArgument(int a, int b, int expectedResult)
{
    var result = _calculator.Max(a, b);

    Assert.That(result, Is.EqualTo(expectedResult));
}
```

## Ignoring a Test

With `[Ignore("Reason to ignore...")]` you can `skip` a test,

```c#
[TestFixture]
public class CalculatorTests
{
    private Calculator _calculator;

    [SetUp]
    public void Setup()
    {
        _calculator = new Calculator();
    }

    [Test]
    [Ignore("Because I want so!")]
    public void Add_WhenCalled_ReturnsSum()
    {
        var result = _calculator.Add(1, 2);

        Assert.That(result, Is.EqualTo(3));
    }

    [Test]
    [TestCase(2, 1, 2)]
    [TestCase(1, 2, 2)]
    [TestCase(1, 1, 1)]
    public void Max_WhenCalled_ReturnTheGreaterArgument(int a, int b, int expectedResult)
    {
        var result = _calculator.Max(a, b);

        Assert.That(result, Is.EqualTo(expectedResult));
    }
}
```

## String Assertions

```c#
[Test]
public void FormatAsBold_WhenCalled_ShouldEncloseTheStringWithStrongElement()
{
    var formatter = new HtmlFormatter();

    var result = formatter.FormatAsBold("abc");

    // Specific
    Assert.That(result, Is.EqualTo("<strong>abc</strong>").IgnoreCase);

    // More general
    Assert.That(result, Does.StartWith("<strong>").IgnoreCase); //All string assertion methods have `IgnoreCase` property
    Assert.That(result, Does.EndWith("</strong>"));
    Assert.That(result, Does.Contain("abc"));
}
```

## Array Assertions

```c#
[Test]
public void GetOddNumbers_LimitIsGreaterThanZero_ReturnOddNumbersUpToLimit()
{
    var result = _calculator.GetOddNumbers(5);

    Assert.That(result, Is.Not.Empty);

    Assert.That(result.Count(), Is.EqualTo(3));

    Assert.That(result, Does.Contain(1));
    Assert.That(result, Does.Contain(3));
    Assert.That(result, Does.Contain(5));

    Assert.That(result, Is.EquivalentTo(new[] { 1, 3, 5 })); //Order is not important. It just checks the existence of the members.

    Assert.That(result, Is.Ordered); //The members should be in ascending order
    Assert.That(result, Is.Unique);  //The members should be unique

}
```

## Object Assertions

```c#
private CustomerController _controller;

[SetUp]
public void Setup()
{
    _controller = new CustomerController();
}

[Test]
public void GetCustomer_IdIsZero_ReturnNotFound()
{
    var result = _controller.GetCustomer(0);

    // NotFound
    Assert.That(result, Is.TypeOf<NotFound>());

    // ActionResult or one of its derivatives
    Assert.That(result, Is.InstanceOf<ActionResult>());
}
```

## Void Assertions

These methods are commands and often change the state of an object. For example the `Log` method in the example below:

```c#
public class ErrorLogger
{
    public string LastError { get; set; }

    public event EventHandler<Guid> ErrorLogged;

    public void Log(string error)
    {
        if (String.IsNullOrWhiteSpace(error))
            throw new ArgumentNullException();

        LastError = error;

        // Write the log to a storage
        // ...

        ErrorLogged?.Invoke(this, Guid.NewGuid());
    }
}
```

To test the `happy path`:

```c#
[Test]
public void Log_WhenCalled_SetTheLastErrorProperty()
{
    var logger = new ErrorLogger();

    logger.Log("a");

    Assert.That(logger.LastError, Is.EqualTo("a"));
}
```

## Throw Exception Assertions

```c#
[Test]
[TestCase(null)]
[TestCase("")]
[TestCase(" ")]
public void Log_InvalidError_ThrowArgumentNullException(string error)
{
    var logger = new ErrorLogger();

    Assert.That(() => logger.Log(error), Throws.ArgumentNullException);
}

```

## Raise Event Assertions

```c#
[Test]
public void Log_ValidError_RaiseErrorLoggedEvent()
{
    var logger = new ErrorLogger();

    var id = Guid.Empty;
    logger.ErrorLogged += (sender, args) => { id = args; };

    logger.Log("a");

    Assert.That(id, Is.Not.EqualTo(Guid.Empty));
}
```

# Mocking

Suppose that we have a `VideoService` class that has one dependency injected to it (if we had a DI framework, we would write it simpler by removing `= null` and `?? new ...`):

```c#
public VideoService(IFileReader fileReader = null)
{
    _fileReader = fileReader ?? new FileReader();
}
```

with one method `string ReadVideoTitle(string path)` which uses `_fileReader`. The interface is:

```c#
public interface IFileReader
{
    string Read(string path);
}

```

We could create `Fake` class (`FakeFileReader`) that implements the above interface and test our `VideoService` class with it but that's not efficient (because we have to define a new fake class if we want to change various outputs!). Instead, we use `Mock Frameworks` such as `Moq`.

## Installing `Moq`

- Search for `Moq` in `NuGet`.
- Add it to the test project.
- restore the test project.

## Using Moq

```c#
private Mock<IFileReader> _fileReader;

private VideoService _videoService;

[SetUp]
public void SetUp()
{
    _fileReader = new Mock<IFileReader>();

    _videoService = new VideoService(_fileReader.Object);
}
```

### Setup method

The example below, indicates that, whenever on the mock object, the `Read` method with `"video.txt"` is called, return `""`:

```c#
[Test]
public void ReadVideoTitle_EmptyFile_ReturnError()
{
    _fileReader.Setup(fr => fr.Read("video.txt")).Returns("");

    var result = _videoService.ReadVideoTitle("video.txt");

    Assert.That(result, Does.Contain("error").IgnoreCase);
}
```

- In the above example we could use `It.IsAny<string>()` instead of `"video.txt"`, if we didn't want to specify the exact value of the parameter of the Read method.
- If the mock object has a `property` that we wanted to setup, we do it like `_fileReader.Setup(fr => fr.Prop1.Returns("Something");`
- We can use `Throws` instead of `Returns`:

```c#
[Test]
public void ReadVideoTitle_EmptyFileName_ReturnError()
{
    _fileReader.Setup(fr => fr.Read("")).Throws<InvalidOperationException>();

    var result = _videoService.ReadVideoTitle("");

    Assert.That(result, Does.Contain("error").IgnoreCase);
}
```

### Verify method

If we want to verify that a method with specific arguments are called, we used `Verify` method on our mock object:

```c#
[Test]
public void ReadVideoTitle_WhenCalled_CallFileReader()
{
    var result = _videoService.ReadVideoTitle("video.txt");

    _fileReader.Verify(fr => fr.Read("video.txt"));
}
```

- In the above example we could use `It.IsAny<string>()` instead of `"video.txt"`, if we didn't want to specify the exact value of the parameter of the Read method.
