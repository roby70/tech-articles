---
title: Unit testing with EF Core async operations
date: 2023-02-16
---
# Unit Testing with EF Core Async Operations 

In the following article I propose a solution for a problem that I faced trying to unit test some business logic code that use async extensions methods to work on collecitons.

## The case of study

I faced the problem to manage unit tests of business logic that uses a repository interface to retrieve a collection of items from the persistence layer (in this case using a EF Core dbcontext).

In the initial implementation I've designed a repository interface that provides access to the database. Something like this:

```csharp
public interface ISomethingRepository {
	Task<Something> GetAsync(int id);

	IQueryable<Something> GetAllItems();
}
```

The `GetAllAsync` method let you retrieve data from your repository and manage it in the business layer, without the overhead of reading all data from the repository.

A typical implementation could be this one:

```csharp
public IQueryable<Something> GetAllItems() {
	var result = _dbContest.SomethingSet;
	return result;
}
```

Now let suppose that you would like to use in the business logic an async operation like `ToListAsync`, eg:

```csharp
public IList<Something> GetPagedData(int pageIndex, int pageSize, Filters filters) {
	var items = _repository;
	items = ApplyFilters(filters, items);
	items = items.GetAllItems()
		.Skip(pageIndex*PageSize)
		.Take(pageSize);
	var result = items.ToListAsync();
}
```

This implementation works fine, the filter are applied with the paging and only the required items are retrieved from the database.

But ... this implementation became hard to test, because the `ToListAsync` is defined as an extension in the  `Microsoft.EntityFrameworkCore` package, and  ...   this implementation assume that under the hood the `IQueryable<Something>` object in reality implements the `IAsyncEnumerable<TSource>` interface.

So why you want to mock the `ISomethingRepository` and provide fake data for `GetAllItems` method it's very hard to provide a  value that works in `ToListAsync`.

In fact mocking the repository class, like in the following example (using [XUnit](https://www.nuget.org/packages/xunit) and [Moq](https://www.nuget.org/packages/Moq) packages), will end in an exception.

```csharp
[Fact]
public async Task GetPagedData_Works() {
	// Arrange
	// ...
	var items = new Something[] { 
	  // some test data ...
	}; 
	var repositoryMock = new MockRepository<ISomethingRepository>();
	repositoryMock.Setup(_ => _.GetAllItems()).Returns(items.AsQueryable());

    // ...
}
```

The following exception will be thrown by the invocation of `ToListAsync`:
```
System.InvalidOperationException : The source 'IQueryable' doesn't implement 'IAsyncEnumerable<Something>'. Only sources that implement 'IAsyncEnumerable' can be used for Entity Framework asynchronous operations.
```


## Lack of separation of concerns

After trying some solution that involve implementation of some class that implement both `IQueryable` and `IAsyncEnumerable` to be used in the test library, I realized that the real problem is that the business logic is not  separated from the persistence layer.

In fact the business logic use  methods defined in  `Microsoft.EntityFrameworkCore`. 
So I tried to find a solution that remove this dependency between the two layers.

## Proposed solution

I ended up with the following solution, that remove the dependency from EF of the business logic and bypass the implicit use of `IAsyncEnumerable`. 
The following code use the [NuGet Gallery | System.Linq.Async 6.0.1](https://www.nuget.org/packages/System.Linq.Async)  package (source code could be found here [dotnet/reactive: The Reactive Extensions for .NET (github.com)](https://github.com/dotnet/reactive)).

I changed the signature of the `GetAllItems` method to return  explicitly `IAsyncEnumerable` instead of `IQueryable`:

```csharp
public interface ISomethingRepository {
   // ...
  IAsyncEnumerable<Something> GetAllItems()
  // ...
}

public class SomethingRepository {
  // ...
  public IAsyncEnumerable<Something> GetAllItems() {
	var items = _dbContest.SomethingSet;
	return result.AsAsyncEnumerable(); // reference to EntityFrameworkCore extension
  }
  // ... 
}

```


Then the business logic was modified using `System.Linq.Async` extensions methods, removing the dependencies from EF:

```csharp
public IList<Something> GetPagedData(int pageIndex, int pageSize, Filters filters) {
	var items = _repository;
	items = ApplyFilters(filters, items);
	items = items.GetAllItems()
		.Skip(pageIndex*PageSize)
		.Take(pageSize);
	var result = items.ToListAsync(); // provided in System.Linq.Async
}
```

Finally the unit test was refactored using the `ToAsyncEnumerable` extension:

```csharp
[Fact]
public async Task GetPagedData_Works() {
	// Arrange
	// ...
	var items = new [] { 
	  // some test data ...
	}; 
	var repositoryMock = new MockRepository<ISomethingRepository>();
	repositoryMock.Setup(_ => _.GetAllItems()).Returns(items.ToAsyncEnumerable());

    // ...
}
```

## Conclusion

I believe that this solution doesn't require any overhead in development and guarantee that EF Core is used only in the layer where it is used. 

On the other side, after some years of developing, I could also assert that rarely the framework/library used to  grant access to the database will change during the lifetime of a project. So any other solution that let unit tests to work with `IQueryable` and `ToListAsync` will be valid too.