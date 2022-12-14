# Configuration
_This content is also available in [the readme](https://github.com/Infocaster/UrlTracker/blob/v10/main/README.md)_

The URL Tracker exposes some configurations in the appsettings.json file. All configurations are optional. A complete configuration may look like this:
```json
{
    "UrlTracker": {
        "IsDisabled": false,
        "IsTrackingDisabled": false,
        "LoggingEnabled": false,
        "IsNotFoundTrackingDisabled": false,
        "AppendPortNumber": false,
        "HasDomainOnChildNode": false,
        "CacheRegexRedirects": true,
        "InterceptSlidingCacheMinutes": 2880,
        "MaxCachedIntercepts": 5000,
        "EnableInterceptCaching": true
    }
}
```

## IsDisabled
| Type | Default | Description |
|:-----|:--------|:------------|
| bool | `false` | Set this value to `true` to completely disable the URL Tracker. The URL Tracker will not intercept any requests nor track any content updates

## IsTrackingDisabled
| Type | Default | Description |
|:-----|:--------|:------------|
| bool | `false` | Set this value to `true` to disable tracking of content changes. The URL Tracker will not automatically create redirects when content is updated

## IsNotFoundTrackingDisabled
| Type | Default | Description |
|:-----|:--------|:------------|
| bool | `false` | Set this value to `true` to disable tracking of Not Found responses.

## EnableLogging
| Type | Default | Description |
|:-----|:--------|:------------|
| bool | `false` | Set this value to `true` to allow the URL Tracker to write logs to the Umbraco native logger. Most logs from the URL Tracker are written at Debug or Verbose level.

## AppendPortNumber
| Type | Default | Description |
|:-----|:--------|:------------|
| bool | `false` | Set this value to `true` to add a port number behind the host component of a redirect url. This setting is ignored when the application is hosted on the default port 80.

## HasDomainOnChildNode
| Type | Default | Description |
|:-----|:--------|:------------|
| bool | `false` | Set this value to `true` if your website has domains configured on pages that are not in the root of the website.

## CacheRegexRedirects
| Type | Default | Description |
|:-----|:--------|:------------|
| bool | `true`  | Set this value to `false` to disable caching of regex redirects. By default, all regex redirects are cached in memory to improve performance.

## InterceptSlidingCacheMinutes
| Type | Default | Description |
|:-----|:--------|:------------|
| int? | `2880`  | Set this value to the time in minutes that all redirects should be cached. By default, all redirects are cached for 2 days. Set to `null` to cache indefinitely.

## MaxCachedIntercepts
| Type | Default | Description |
|:-----|:--------|:------------|
| long | `5000`  | Set this value to the amount of intercepts that should be cached by the UrlTracker. This not only includes redirects, but also 200 OK responses, 410 GONE responses and 404 NOT FOUND responses.

## EnableInterceptCaching
| Type | Default | Description |
|:-----|:--------|:------------|
| bool | `true`  | Set this value to `false` to completely disable redirect caching.

_Continue on to learn [how to extend the URL Tracker](./Extending)._

<a href="https://infocaster.net">
<img align="right" height="200" src="https://github.com/Infocaster/.github/blob/cba580027a6761844ddab7267f85debe31e96f1a/assets/Infocaster_Corner.png?raw=true">
</a>