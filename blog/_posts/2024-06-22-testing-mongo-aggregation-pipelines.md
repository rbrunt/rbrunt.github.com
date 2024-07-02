---
layout: post
title: Testing Mongo Aggregation Pipelines with EphemeralMongo
description: Why and how to test MongoDB Aggregation Pipelines in .NET using the EphemeralMongo library & xUnit
published: true
---

Recently while working on a project at work, I wanted to test if the aggregation pipeline at the heart of an API was working as expected. While writing up some documentation to summarise some of the business logic for future reference, I realised that there might be some edge cases that didn’t behave as we would have wanted, potentially leading to stale data being shown to the user.

In this situation, we would normally want to write unit tests to carefully probe these edge cases and adjust the code we’ve written until it behaves as we expect. This is trivial when the code you want to test is in your backend:  you just write a new test in your test suite. It wasn’t so easy in this case though: although the mongo aggregation pipeline was _defined_ in our .NET code (in the form of a `BSONDocument` object) it was actually being _executed_ on the database, meaning we couldn’t put it under test using the existing testing infrastructure. This led me to investigating various options for unit testing aggregation pipelines, which eventually led me to the [EphemeralMongo](https://github.com/asimmon/ephemeral-mongo){:target="_blank"} project.

This blog post will discuss my experiences of using EphemeralMongo to unit test the Procuration Fee Management API, and hopefully point you in the right direction if you’re interested in implementing something similar in your .NET code.

## The Problem

[Aggregation Pipelines](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/){:target="_blank"} are a great way to combine information from multiple documents or even documents from different collections inside a mongo database. Offloading those operations to the database makes sense: running data intensive operations close to where the data resides improves both application responsiveness and overall resource usage. Rather than pulling all those disparate entities down into your application over several individual calls, why not send the database a list of instructions and let it return just the information you need?

Aggregation pipelines are often used in batch jobs to summarise data in a database, but they’re also often used for more mundane aggregations. In one system, aggregation pipelines were used to summarise a series of events into the current state of an entity, and in another, they encapsulate business logic around duplicate record detection and running searches. In both cases these aggregation pipelines are defined in C# backend code but they’re really just data structures we send to the database for execution, much like a SQL query. This means that they’re not executed by the .NET runtime, and so we can’t ‘just’ test them with a normal unit test.


Aggregation pipelines can be defined using LINQ syntax…

```c#
_mongoCollection
.Aggregate()
.Match(filter)
.SortByDescending(x => x.LastUpdatedAt)
.Group(y => y.RecordId, z => new InterestingEntity
{
    Id = z.First().Id,
    RecordId = z.First().RecordId,
    FirstName = z.First().FirstName,
    LastName = z.First().LastName,
    SomeDate = z.First().SomeDate,
    Status = z.First().Status,
    CreatedBy = z.First().CreatedBy,
    CreatedAt = z.First().CreatedAt,
    LastUpdatedBy = z.First().LastUpdatedBy,
    LastUpdatedAt = z.First().LastUpdatedAt,
})
.SortByDescending(x => x.LastUpdatedAt);
```
LINQ will compile this statement down to a BSONDocument object at runtime, ready to send to the database for execution.

…or by manually creating a series of BSON Documents:

```c#
_mongoCollection
.Aggregate(
new BsonDocument() {
// Get potential list of duplicates:
new BsonDocument("$match",
    new BsonDocument
    {
        { "FirstName", entity.FirstName },
        { "Status", new BsonDocument("$eq", entity.Status) }
    }),
// Lookup full series of edits for each recordId:
new BsonDocument("$lookup",
    new BsonDocument
    {
        { "from", "entity_collection" },
        { "localField", "RecordId" },
        { "foreignField", "RecordId" },
        { "as", "result" }
    }),
// unwind, replaceRoot to get back to flat list of documents:
new BsonDocument("$unwind", new BsonDocument("path", "$result")),
new BsonDocument("$replaceRoot", new BsonDocument { { "newRoot", "$result" } }),
// etc...
});
```

Mongo provide some nice tooling for working with aggregation pipelines in [Compass](https://www.mongodb.com/products/compass){:target="_blank"}, their desktop GUI. Compass allows you to paste in some raw BSON and visualise the various stages of your pipeline using real data stored in your database. You can interrogate the inputs and outputs of each stage, and make changes to ensure it’s behaving as you want. This is great to help you reason about what is going on. Once you’re happy with your pipeline definition, you can even [export it to your favourite language](https://www.mongodb.com/docs/compass/current/export-pipeline-to-language/) (like C#), ready to be pasted into your codebase.

These are great features, but they have two major shortcomings: Firstly, though you can paste *BSON* into the pipeline editor, you can’t paste C# code. This means you have to manually translate your code into BSON just to start testing which is error prone, especially on large pipelines. Secondly, targetting edge cases is difficult in Compass unless you have exactly the data you need already in your database. If you don’t, you need to jump into some other tool or import it manually so your data is in a known state for testing.

Together, this makes Compass a great tool for building your pipeline definition from the start or for adhoc testing against real data, but it’s not set up for the quick, highly targetted and repeatable testing that unit tests excel at.


## How does EphemeralMongo help when testing aggregation pipelines?

EphemeralMongo is a set of NuGet packages that allow you to spin up ephemeral, isolated MongoDB databases for testing. The package includes the full MongoDB executable, so you’re running a real Mongo instance for your tests rather than just an approximation. It supports a wide range of .NET framework and .NET core versions, and getting it integrated into your existing unit test infrastructure is really simple.

Once you’ve wired it up, you can start running unit tests (I suppose these are integration tests now, if we’re being pedantic) against your aggregation pipelines by passing in the mongo connection details generated for you to your repositories. Though the code is still running in a mongo database rather than the .NET runtime, it’s so tightly integrated that you won’t really notice: the impact of adding this to the startup time of my unit tests was about 3-5 seconds on my machine, which is much less than the amount of time it takes to build the solution in Visual Studio anyway.

Unit testing pipelines with EphemeralMongo addresses the two major shortcomings of other testing approaches: Firstly, you don’t need to do any error prone manual translation from .NET into the BSON for your testing: you can leave that to the same driver that will be running the queries in your application in production. Secondly, you can now define your input data like any other unit test to probe edge cases and catch regressions in a quick and repeatable way. Shortly after integrating EphemeralMongo into one project, I was able to find and fix a subtle bug that I had missed in my aggregation pipeline. This is something that would have taken me much, much more time to do with other tooling, and now that the tests are written, it will be obvious if a future change cause a regression.

## How do I set it up?

Hopefully you’re now sold enough on EphemeralMongo to want to integrate it into the test suite for your application. Though the details might be slightly different for your application, the broad steps will be similar, so you should be able to translate them into your context without too much thinking.

First, we need to install the NuGet Package into our test project: [check the Downloads section](https://github.com/asimmon/ephemeral-mongo#downloads) for a link to the nuget package for your version of mongo, but we’ll be using EphemeralMongo6 (which targets MongoDB 6.0.7) in this example: https://www.nuget.org/packages/EphemeralMongo6/


Once our nuget package is installed, we need to wire up our setup logic so that we can create our mongo instance for our tests. We want a single mongo instance per test run, shared across all the tests (so that we don’t use up lots of resources on the computer), but we’ll run each test in its own isolated mongo database so that we know that they’re independent.

There are [a few ways of sharing context between tests in xUnit](https://xunit.net/docs/shared-context), but the method we want is the ‘Collection Fixtures’ method. This allows a single database fixture to be shared across all our test classes, rather than creating a new instance per class or per test. To do this, we need to create a new class in our test project to do this wiring:

```c#
// From: MongoDatabaseFixture.cs
using EphemeralMongo;
using Xunit;

namespace SomeTestProject.TestHelpers;

public class MongoDatabaseFixture : IDisposable
{
    private readonly MongoRunnerOptions _mongoRunnerOpts = new() { KillMongoProcessesWhenCurrentProcessExits = true };

    public IMongoRunner runner { get; private set; }

    public MongoDatabaseFixture()
    {
        runner = MongoRunner.Run(_mongoRunnerOpts);
    }

    public void Dispose()
    {
        runner.Dispose();
    }
}

/// <summary>
/// Empty class used to manage an XUnit 'collection context' so that we only create a single
/// 'EphemeralMongo' instance per test run and can use it across different test classes.
/// This class is never actually instantiated, it's just used by XUnit as config during test setup.
/// </summary>
[CollectionDefinition("Mongo Database Tests")]
public class MongoDatabaseCollection : ICollectionFixture<MongoDatabaseFixture>
{
}
```

Here, our fixture handles the creation and destruction of the ephemeral mongo database, and we have a separate utility class to attach our CollectionDefinition to.

Now, whenever we want to write a test against our mongo database we just need to add the `[Collection("Mongo Database Tests")]` attribute to the class and a parameter in the constructor to take the test fixture (so we can actually access the database!), and xUnit will sort the rest out.

Once we have our fixture in our test class we can use it to do any setup we want. Regardless, we’ll want to generate a new database per test. In xUnit, you do this in the constructor of the test class. This gets run once per test, before the test is run. The example below generates a new database for the test that’s running (with a GUID as its name), ensures the collection we’re expecting is already there, then injects the database connection details into the repository (our system under test) ready for testing:

```c#
// using statements omitted for brevity.
namespace SomeTestProject;

[Collection("Mongo Database Tests")]
public class SomeMongoPipelineTests
{
    protected readonly IMongoDatabase _database;
    protected readonly IMongoCollection<SomeEntity> _collection;
    protected readonly SomeEntityRepository _sut;
    
    public SomeMongoPipelineTests(MongoDatabaseFixture fixture)
    {
        var dbName = Guid.NewGuid().ToString().Replace("-", "");
        var options = Options.Create(new MongoDbSettings()
        {
            ConnectionString = fixture.runner.ConnectionString,
            Database = dbName,
            CollectionName = "entity_collection" // Adjust depending on your requirements
        });

        _database = new MongoClient(fixture.runner.ConnectionString).GetDatabase(dbName);
        _collection = _database.GetCollection<SomeEntity>(options.Value.CollectionName); // We'll need access to this in our tests for some setup actions

        _sut = new SomeEntity(options, _database);
    }
    
    //Our [Fact]s and [Theory]s go here...
}
```

If you have some large or complex data to import into the database, you would probably want to hook that into the constructor too: EphemeralMongo provides an `Import()` method that takes json file paths as an input and will bulk insert data into your collection.

Now we’ve done all our setup, we’re ready to write some tests! In my case, I ended up moving a lot of that boilerplate code to a base class, which all my test classes inherit from, just so that I don’t have to copy it out every time, then I just went about writing my tests as I usually would. The result is refreshingly boring:

```c#
// Obviously, this code would normally live in a class:
[Fact]
public async Task ExactCopyIsFlaggedAsDuplicate()
{
    // Arrange
    var now = DateTime.UtcNow;
    var entity1 = GenerateSomeEntity(now); // A helper method to generate data
    // Accessing the mongo collection directly to do some additional setup of our data for this specific test:
    await _collection.InsertOneAsync(entity1);

    var entity2 = entity1 with
    {
        Id = ObjectId.GenerateNewId().ToString(),
        RecordId = Guid.NewGuid().ToString()
    };

    // Act
    var duplicateDetected = await _sut.CheckForDuplicates(entity2); // This was run on MongoDB, not that you can tell from here!

    // Assert
    duplicateDetected.Should().BeTrue();
}
```
