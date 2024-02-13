# Version 

## Why You Need API Versioning

API versioning allows your API to evolve independently from the clients using it.

Introducing breaking changes to your API is a bad user experience. API versioning gives you a mechanism to avoid exposing breaking changes to clients. Instead of making a breaking change, you introduce a new API version.

## What's the definition of a breaking change?

This isn't an exhaustive list, but a few examples of breaking changes are:

- Removing or renaming APIs or API parameters
- Changing the behavior of existing APIs
- Changing the API response contract
- Changing the API error codes


You can decide what a breaking change means for your API. For example, adding a new field to the response doesn't have to be a breaking change.

Let's see how to implement API versioning.

Let's start by installing two NuGet packages that we'll need to implement API versioning:

1. Asp.Versioning.Http
2. Asp.Versioning.Mvc.ApiExplorer

This allows us to call AddApiVersioning and provide a delegate to configure the ApiVersioningOptions.

```
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1);
    options.ReportApiVersions = true;
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new HeaderApiVersionReader("X-Api-Version"));
}).AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'V";
    options.SubstituteApiVersionInUrl = true;
});
```

Here's the explanation for the ApiVersioningOptions properties:

- DefaultApiVersion - Sets the default API version. Typically, this will be v1.0.
- ReportApiVersions - Reports the supported API versions in the api-supported-versions response header.
- AssumeDefaultVersionWhenUnspecified - Uses the DefaultApiVersion when the client didn't provide an explicit version.
- ApiVersionReader - Configures how to read the API version specified by the client. The default value is QueryStringApiVersionReader.


The AddApiExplorer method is helpful if you are using Swagger. It will fix the endpoint routes and substitute the API version route parameter.

​

## Different Types of API Versioning

The most common ways to implement API versioning are:

URL versioning: https://localhost:5001/api/v1/workouts
Header versioning: https://localhost:5001/api/workouts -H 'X-Api-Version: 1'
Query parameter versioning: https://localhost:5001/api/workouts?api-version=1

There are a few other ways to implement API versioning. For example, using the accept or content-type headers. (But they aren't used often.)

The Asp.Versioning.Http library has a few IApiVersionReader implementations:

UrlSegmentApiVersionReader
HeaderApiVersionReader
QueryStringApiVersionReader
MediaTypeApiVersionReader




Microsoft's API versioning guidelines  suggest using URL or query string parameter versioning.




Versioning Controllers

To implement API versioning in ASP.NET controllers, you have to decorate the controller with the ApiVersion attribute.

The ApiVersion attribute allows you to specify which API versions that WorkoutsController supports. In this case, the controller supports both v1 and v2. You use the MapToApiVersion attribute on the endpoints to specify the concrete API version.

The route parameter v{v:apiVersion} lets you specify the API version using v1 or v2 in the URL.

[ApiVersion(1)]
[ApiVersion(2)]
[Route("api/v{v:apiVersion}/workouts")]
public class WorkoutsController : ControllerBase
{
    [MapToApiVersion(1)]
    [HttpGet("{workoutId}")]
    public IActionResult GetWorkoutV1(Guid workoutId)
    {
        return Ok(new GetWorkoutByIdQuery(workoutId).Handle());
    }

    [MapToApiVersion(2)]
    [HttpGet("{workoutId}")]
    public IActionResult GetWorkoutV2(Guid workoutId)
    {
        return Ok(new GetWorkoutByIdQuery(workoutId).Handle());
    }
}
​

Deprecating API Versions

If you want to deprecate an old API version, you can set the Deprecated property on the ApiVersion attribute. The deprecated API versions will be reported using the api-deprecated-versions response header.

[ApiVersion(1, Deprecated = true)]
[ApiVersion(2)]
[Route("api/v{v:apiVersion}/workouts")]
public class WorkoutsController : ControllerBase
{
}
​

Takeaway

API versioning is one of the best practices for designing modern APIs. Consider implementing API versioning from the first release. This makes it easier for clients to support future API versions. And it gets your team used to managing breaking changes and versioning the API.

You can use the Asp.Versioning.Http library to add API versioning in ASP.NET Core. Define the supported API versions, and start using them in your endpoints.

Remember to agree as a team what represents a breaking change. This should be well documented in the team's API design guidelines.