# Extending
## Create, search and update redirects in code
Use the public api to create, search and update redirects in code:

```csharp
using UrlTracker.Core;
using UrlTracker.Core.Models;

public class MyService
{
    private readonly IUmbracoContextFactory _umbracoContextFactory;
    private readonly IRedirectService _redirectService;

    public MyService(IUmbracoContextFactory umbracoContextFactory, IRedirectService redirectService)
    {
        _umbracoContextFactory = umbracoContextFactory;
        _redirectService = redirectService
    }

    public async Task MyMethod()
    {
        using var cref = _umbracoContextFactory.EnsureUmbracoContext();

        // Create a new redirect
        Redirect redirect = new Redirect
        {
            SourceUrl = "/home/lorem/ipsum"
            TargetRootNode = cref.UmbracoContext.Content.GetById(1234),
            TargetNode = cref.UmbracoContext.Content.GetById(1235),
            TargetStatusCode = HttpStatusCode.Found,
            Culture = "en-US"
        }

        // Add the redirect and reassign the variable. The new value has an id assigned.
        //    If the redirect is invalid, this method will throw an ArgumentException.
        //    Check the inner exception for details.
        redirect = await _redirectService.AddAsync(redirect);

        // Change the redirect
        redirect.TargetNode = null;
        redirect.TargetUrl = "https://example.com/";

        // Update the redirect and reassign the variable
        //    If the redirect is invalid, this method will throw an ArgumentException.
        //    Check the inner exception for details.
        redirect = await _redirectService.UpdateAsync(redirect);

        // Search for redirects
        RedirectCollection searchResults = await _redirectService.GetAsync(skip: 0, take: 10, query: "homepage");
    }
}
```

## Custom request intercept filter
Implement a custom request intercept filter to filter out incoming requests. You can prevent particular urls from being processed by the URL Tracker, thus improving performance. Following is an example of a custom url filter that would filter out all api urls so they won't be processed by the URL Tracker:

```csharp
using UrlTracker.Core.Domain.Models;
using UrlTracker.Web;
using UrlTracker.Web.Processing;

public class ApiRequestFilter : IRequestInterceptFilter
{
    public ValueTask<bool> EvaluateCandidateAsync(Url url)
    {
        // Return false to stop processing of this url. Return true to continue processing.
        //    If any filter returns false, the URL Tracker will not process the request.
        return new ValueTask<bool>(url.Path?.StartsWith("/api") is not true);
    }
}

public class MyComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        // Add the filter to the collection
        builder.RequestInterceptFilters()
            .Append<ApiRequestFilter>();

        // Remove any other filter from the collection if you want
        builder.RequestInterceptFilters()
            .Remove<UrlReservedPathFilter>();
    }
}
```

## Custom client error filter
Implement a custom client error filter to filter out requests with client errors. You can prevent particular urls from being registered in the client error overview, so they won't be displayed on the dashboard. Following is an example of a custom url filter that would filter out all client errors to javascript source maps:

```csharp
using UrlTracker.Web;
using UrlTracker.Web.Processing;

public class JsSourceMapClientErrorFilter : IClientErrorFilter
{
    public ValueTask<bool> EvaluateCandidateAsync(UrlTrackerHandled notification)
    {
        // Return false to stop prevent registration of this client error. Return true to continue registration
        //    If any filter returns false, the URL Tracker will not register the client error.
        return new ValueTask<bool>(notification.Url.Path?.EndsWith(".js.map") is not true);
    }
}

public class MyComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        // Add the filter to the collection
        builder.ClientErrorFilters()
            .Append<JsSourceMapClientErrorFilter>();

        // Remove any other filter from the collection if you want
        builder.ClientErrorFilters()
            .Remove<ConfigurationClientErrorFilter>();
    }
}
```

**NOTE: At the time of writing, the URL Tracker only supports registration of 404 NOT FOUND results as client errors**

## Custom matching and handling
Should you find that the URL Tracker matching services are not enough for your needs, the URL Tracker provides means to extend matching and handling logic. The process takes two steps, though more customization is possible if required

### Matching
The first step is to create a custom matching component. In the URL Tracker, these are called 'handlers'.
```csharp
using UrlTracker.Core;
using UrlTracker.Core.Domain.Models;
using UrlTracker.Core.Intercepting;
using UrlTracker.Core.Intercepting.Models;

public class MyCustomInterceptor : IInterceptor
{
    public ValueTask<ICachableIntercept?> InterceptAsync(Url url, IReadOnlyInterceptContext context)
    {
        // Read properties from the url and the context
        var culture = context.GetCulture();
        
        // Whatever you return here is what will later be handled by the handlers
        // You could:
        //  1) return a familiar model. Familiar models already have handlers and won't require you to define a handler yourself. Familiar models are: UrlTrackerShallowRedirect, UrlTrackerShallowClientError and CachableInterceptBase.NullIntercept
        return new ValueTask<ICachableIntercept?>(new CachableInterceptBase<UrlTrackerShallowRedirect>(new UrlTrackerShallowRedirect()));

        //  2) return your own model. You'll have to implement a handler later to handle your own model
        //     Either return CachableInterceptBase with an instance of your model or implement ICachableIntercept yourself.
        //     As the name suggests: your model must be cachable. You shouldn't return IPublishedContent here for example.
        return new ValueTask<ICachableIntercept?>(new CachableInterceptBase<MyCachableModel>(new MyCachableModel()));

        //  3) return null. return null to indicate that your handler couldn't match the incoming url. The URL Tracker will continue the search with other handlers
        return new valueTask<ICachableIntercept?>(null);
    }
}

public class MyComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        // Add the handler to the collection
        builder.Interceptors()
            .Append<MyCustomInterceptor>();

        // Remove any other handlers from the collection if you want
        builder.Interceptors()
            .Remove<RegexRedirectInterceptor>();

        // Interceptors are evaluated in order.
        //    You can also insert your handler before another
        builder.Interceptors()
            .InsertBefore<NoLongerExistsInterceptor, MyCustomInterceptor>();
    }
}
```

### Handling
The next step is to handle the intercept

```csharp
using UrlTracker.Web.Processing;

public class MyInterceptHandler : ResponseInterceptHandlerBase<MyInterceptModel>
{
    protected override async ValueTask HandleAsync(RequestDelegate next, HttpContext context, MyInterceptModel intercept)
    {
        // From here you're acting as a middleware and can decide what to do with the request, based on the intercept model.
        // You can choose to let the request through:
        await next(context);

        // You can also do something else:
        context.Response.StatusCode = 418;

        // It's all up to you what happens now... continue, change, return, etc.
    }
}

public class MyComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        // Add the handler to the collection
        builder.ResponseInterceptHandlers()
            .Append<MyInterceptHandler>();

        // Remove any other handlers from the collection if you want
        builder.ResponseInterceptHandlers()
            .Remove<RedirectResponseInterceptHandler>();

        // Interceptors are evaluated in order.
        //    You can also insert your handler before another
        builder.ResponseInterceptHandlers()
            .InsertBefore<NullInterceptHandler, MyInterceptHandler>();
    }
}
```

<a href="https://infocaster.net">
<img align="right" height="200" src="https://github.com/Infocaster/.github/blob/cba580027a6761844ddab7267f85debe31e96f1a/assets/Infocaster_Corner.png?raw=true">
</a>