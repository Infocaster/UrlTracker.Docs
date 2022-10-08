# Getting Started
_This content is also available in [the readme](https://github.com/Infocaster/UrlTracker/blob/v10/main/README.md)_

## Requirements
For the URL Tracker to function properly, you need to be connected with a SQL Server database. NOTE: Certain functions are known not to work on SQL Compact Edition and we don't officially support it.  
The URL Tracker should work on Umbraco Cloud.

## Setup
The URL Tracker is available via NuGet. Visit [the URL Tracker on NuGet](https://www.nuget.org/packages/UrlTracker/) for instructions on how to install the URL Tracker package in your website.
Once installed, you'll have to actually use it in your request pipeline (Often found in the file `Startup.cs`). For the best performance, you should insert the UrlTracker as high in the pipeline as possible. The UrlTracker requires an instantiated umbraco context, so make sure it is inserted after the umbraco context is initialized. We recommend that you insert the UrlTracker like this:
```csharp
app.UseUmbraco()
    .WithMiddleware(u =>
    {
        u.UseBackOffice();
        u.UseWebsite();

        // Insert behind 'UseWebsite' to ensure the existance of an UmbracoContext
        u.UseUrlTracker();
    })
```
Now build your project and you should be ready to make your visitors happy!

_Continue on to learn [how to configure the URL Tracker](./Configuration)._
<a href="https://infocaster.net">
<img align="right" height="200" src="https://github.com/Infocaster/.github/blob/cba580027a6761844ddab7267f85debe31e96f1a/assets/Infocaster_Corner.png?raw=true">
</a>