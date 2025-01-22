---
layout: post
date: 2025-01-22
categories: tips, c#
title: Result class for Dotnet
---

Functions in dotnet can only return one value but there are times when you want the option to be able to return an error result. You could throw an exception and use a `try catch` to catch the error, but that feels a little overkill for more cases.

## Result class ##

A more elegant solution is to use a Result class that can be either the result of the function or an error result. A simple implementation of such a class is: 

```
public class Result<TError, TValue> where TError : IError
{
    private readonly TError? _error;
    private readonly TValue? _value;

    public Result(TValue value) => _value = value;
    public Result(TError error) => _error = error;

    public bool IsSuccess => _value is not null;

    public TValue? Value => _value ?? default;
    public TError? Error => _error;
}
public interface IError
{
    string Message { get; init; }
}
```

This class could be used in a function by defining the return value as `Result<GenericError,ReturnData>` where `GenericError` is an implementation of `IError` and `ReturnData` is whatever class that you would normally return from the function.

Because there is a constructor for both `TError` and `TValue` you can create a new `Result` class by 
passing either a error or the result into the constructor.

```
return new Result<GenericError, ReturnData>(new GenericError("Error"));
// or
return new Result<GenericError, ReturnData>(new ReturnData { Message = "Success" });
```

## Implicit constructors ##

This can be made more simple by adding implicit operators to the `Result` class to automatically convert return values to the return type.

```
    public static implicit operator Result<TError, TValue>(TValue value) => new(value);

    public static implicit operator Result<TError, TValue>(TError error) => new(error)
```

These allow you to remove the `Result` type from the return statement, so the return statements can look like

```
return new GenericError("Error");
// or
return new ReturnData { Message = "Success" };
```

## Processing result ##

But how do you check the result? You can check the success property on the returned value, processing the error if it is false or getting the value if it true.

```
if (!result.IsSuccess)
{
    Console.WriteLine($"Error is {result.Error!.Message}");
}
else 
{
    // work with result.Value
}
```

But we can do better than that, if we add a match function to the result class we can make the error check a bit more declarative

```
public TResult Match<TResult>(Func<TError, TResult> failure, Func<TValue, TResult> success) =>
    IsSuccess
        ? success(Value ?? throw new NullReferenceException())
        : failure(Error ?? throw new NullReferenceException());
```

And it can be used like

```
var processResult = notError.Match<string>(
    x =>
    {
        Console.WriteLine($"Error is {x.Message}");
        return "Error";
    },
    x =>
    {
        Console.WriteLine($"Data is {x.Message}");
        return x.Message;
    });
```

## Full class ##

```
public class Result<TError, TValue> where TError : IError
{
    private readonly TError? _error;
    private readonly TValue? _value;

    public Result(TValue value) => _value = value;
    public Result(TError error) => _error = error;

    public bool IsSuccess => _value is not null;

    public TValue? Value => _value ?? default;
    public TError? Error => _error;

    public static implicit operator Result<TError, TValue>(TValue value) => new(value);

    public static implicit operator Result<TError, TValue>(TError error) => new(error);

    public TResult Match<TResult>(Func<TError, TResult> failure, Func<TValue, TResult> success) =>
        // Here we can throw exception because it's a failure case, not an error
        // It's normally impossible to get an exception from here, but compiler needs it
        IsSuccess
            ? success(Value ?? throw new NullReferenceException())
            : failure(Error ?? throw new NullReferenceException());
}
```

## Additional tip ##

It can be annoying to have to put the full type in all the function declarations, but there is a solution by using a global using type alias statement. In a file called `GlobalUsings.cs` you can define an alias like

```
global using StandardReturn = Helpers.Result<Helpers.GenericError, Helpers.ReturnData>;
```

then you can use the type `StandardReturn` anywhere you had previously used `Result<GenericError,ReturnData>`