<!---
    ::::
    ::  Author: Bryan McCoy
    ::  Title: Why do you TDD? 
    ::  Date: 02/01/2019
    ::  Tags: TDD, .Net Core, Test driven development, unit test
    ::  Live: Yes
    ::::
--->

## Why do you TDD?

So, the app is coming along now.  Because of a previous post, we now have a basic structure for configuring the app.  But I still feel like we are missing something.  Something very important to me.  Unit Tests!  I am a fan of TDD (test driven development).  You couldn’t tell that form this app, well, because there aren’t any.  I have a bad habit of not writing test in my proof of concept apps.  However, TDD is something I practice in my professional work.

So, why do I TDD?  I’m not going to get preachy here, if you don’t write unit test then that is fine.  I am not a traditionalist, in a sense.  I write unit test to make my life easier.  I don’t like having to run my application whenever I want to test if some logic actually does what it is supposed to do.  Also, I don’t want to have to set up my data to create a certain scenario for the same reason.  It honestly helps me write cleaner code.  Basically if my class, method, function, or whatever isn’t testable, then it probably isn’t written well. 

Now let's get to the code.  We will start with getting our testing framework setup, then we will start writing some simple tests, and discuss additional tools that are available to us.  We want the barrier to entry to be as low as possible.  People don’t tend to adopt things they see as a nuisance. 

<!--- End Preview --->

## WHERE TO PUT OUR TEST
---
Well, this one is fairly obvious.  We are going to create a “tests” folder next to our "src" folder.  That way we can keep all of our tests together and organized.  Inside of that test folder we will add our CoreBlogger.Tests project.  We will then create some folders to store our test based off of our project structure.  So let's do that!

So we will create and navigate to the tests folder and run the following command.

``` c#
dotnet new xunit -o CoreBlogger.Tests
```

After that we will go to the “CoreBlogger.Test” folder and run “dotnet test” and there we have it!  A single passing unit test.

![Passing Test](https://i.imgur.com/P534SMh.png)

So quick recap.  We used the .Net Cli to generate a new test project and “output” (-o) it into the CoreBlogger.Tests folder. Once we did that we went into the newly created folder and used the .Net Cli to run our unit tests.  It was that easy.

Now we have something we can start to work with!

## ACTUALLY TESTING SOMETHING
---
So let's take a look.

![Test folder](https://i.imgur.com/tS8x9L5.png)

As you can see we have a tests folder, with a CoreBlogger.Tests folder, a csproj file, and finally a single UnitTest1.cs class. If we open up our csproj we will see the initial tools and frameworks that are required to run our unit test.

``` xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>

    <IsPackable>false</IsPackable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="15.8.0" />
    <PackageReference Include="xunit" Version="2.3.1" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.3.1" />
  </ItemGroup>

</Project>
```
So we have the sdk, xunit library, and the xunit.runner.visualstudio package.  Next we will look at the UnitTest1.cs (which we will remove here shortly).

``` c#
using System;
using Xunit;

namespace CoreBlogger.Tests
{
    public class UnitTest1
    {
        [Fact]
        public void Test1()
        {

        }
    }
}
```

Again, nothing fancy here.  Just a plain class with an Xunit reference, System reference, and a class with a single method decorated with a “Fact” attribute.  

> I found this link https://xunit.github.io/docs/comparisons that has a wonderful class attribute compassion if you are familiar with other frameworks.  I typically write tests using MSTest, so I found this very helpful.

So we have see this test pass, lets make it fail… You know, for science.  We will change the UnitTest1 class to look like:

``` c#
using System;
using Xunit;

namespace CoreBlogger.Tests
{
    public class UnitTest1
    {
        [Fact]
        public void Test1()
        {
            Assert.Equal(1, 2);
        }
    }
}
```
Silly, I know.  We will do better, we are just messing around with the framework right now.  As you can see from the image below, the test did fail.  The output tells us what the actual value that was passed in is and what it should be.  Now lets delete this file and actually test something with purpose. 

![Failed Test Image](https://i.imgur.com/ce4tW7n.png)

## GETACTIVEBLOGENTRIESHANDLER
---
Well, that’s much harder to read then I would have liked (GetActiveBlogEntriesHandler).  Anyway, let’s move on. The first and only class we are going to test is in this post is GetActiveBlogEntriesHandler.cs.  Not normally the place I would start, but it will be a good example.

So I’ve made a test class in my current test project.  CoreBlogger.Tests > Core > Handlers > GetActiveBlogEntriesHandlerTests.cs.  So here you can see the basic file structure.  Now we test!

We have the class now we need to test something.  As of now we have two methods, the constructor and Handle.  We want to test both, they both get called when the app uses this class.

> In my exploration of xUnit I see that they do not like the “Setup” and “Teardown” of tests.  Since we are using their framework will won’t use that, we will just setup and teardown in our test methods.  They explain why, but we won’t get into that right now.

## CONSTRUCTOR TEST
---

So we write our first test.  We want to make sure that when we construct a new handler we actually get a new handler.  Easy enough, however… The constructor requires two classed.  We will need an instance of ILogger and ICachedGitHubEntryProvider.  So, let’s find a tool to help us “mock” those.  We are not writing integration tests, so we don’t need our actual implementations of those classes.  We want a fake one that we can control so that way we can test specific behaviors. Enter Moq!

So now we have: 

``` c#
namespace CoreBlogger.Tests.Core.Handlers
{
    public class GetActiveBlogEntriesHandlerTests
    {
        [Fact]
        public void ConstructorTest()
        {
            var logger = new Mock<ILogger<GetActiveBlogEntriesHandler>>();
            var cachedGitHubEntryProvider = new Mock<ICachedGitHubEntryProvider>();
            var result = new GetActiveBlogEntriesHandler(logger.Object, cachedGitHubEntryProvider.Object);

            Assert.NotNull(result);
        }
    }
}
```

We have moq create our mocked classes/services and we can now construct our class, success!  We run dotnet test and we see that we have one passing test.

![Passing Unit test](https://i.imgur.com/W5cfTHD.png)

One passing test down, now onto the next.

## HANDLE TEST
---

Now we need to make sure that when the Handle method is run that we get the proper result.  So first we need a GetActiveBlogEntriesHandler class, and with that we need our logger and provider.  So we will mock them again.  But this time we are going to use the Mocked version of our CachedGitHubtEntryProvider to return  our expected result.  Our expected result is just an active blog post, we want to make sure the other logic in the handler doesn’t remove it.

``` c#
[Fact]        
public void HandleReturnsListSuccess()
{
    var logger = new Mock<ILogger<GetActiveBlogEntriesHandler>>();
    var cachedGitHubEntryProvider = new Mock<ICachedGitHubEntryProvider>();
    var handler = new GetActiveBlogEntriesHandler(logger.Object, cachedGitHubEntryProvider.Object);

    var request = new GetActiveBlogEntriesQuery();
    var cancelationToken = new System.Threading.CancellationToken();

    var gitHubEntry = new GitHubBlogEntry();
    gitHubEntry.SetContent(activeContent);

    var expected = new List<GitHubBlogEntry>{gitHubEntry};

    cachedGitHubEntryProvider.Setup(x => x.GetEntries()).ReturnsAsync(expected);

    var actual = handler.Handle(request, cancelationToken).Result;

    Assert.Equal(expected, actual);
}
```
So a quick break down of what we have here.  First we need to get our mocked services for our handler to use.  Then we need to create the handler itself.  Once we have that we create the objects used by the “Handle” method, nothing fancy in here.  Next we set up our expected result, then tell our cachedGitHubEntryProvider to return our expected result.  Now we call our method and compare our results.

In the next test I will actually get our provider to return a list with a blog post that is not active.  That way we can ensure that our handler logic removes the not active blog post and returns us an empty list.

``` c#
[Fact]        
public void HandleReturnsEmptyListSuccess()
{
    var logger = new Mock<ILogger<GetActiveBlogEntriesHandler>>();
    var cachedGitHubEntryProvider = new Mock<ICachedGitHubEntryProvider>();
    var handler = new GetActiveBlogEntriesHandler(logger.Object, cachedGitHubEntryProvider.Object);

    var request = new GetActiveBlogEntriesQuery();
    var cancelationToken = new System.Threading.CancellationToken();

    var gitHubEntry = new GitHubBlogEntry();
    gitHubEntry.SetContent(notActiveContent);

    var expected = new List<GitHubBlogEntry>();

    cachedGitHubEntryProvider.Setup(x => x.GetEntries()).ReturnsAsync(new List<GitHubBlogEntry>{gitHubEntry});

    var actual = handler.Handle(request, cancelationToken).Result;
    
    Assert.Equal(expected, actual);
}
```

And as we can see from the image below we now have 3 tested scenarios and a basic structure for us to begin backfilling the rest of our tests!

![Surccessful Unit Tests](https://i.imgur.com/Hj3pp2V.png)

Well, I beleive that is all for now. Happy testing!
