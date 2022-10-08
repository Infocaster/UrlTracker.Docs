# Frequently asked questions
## Why does my redirect not work?
There are many reasons why a redirect might not work. To get more insight in what the URL Tracker is doing, follow these steps:
 1. Set loglevel to Debug or lower
 2. Visit the url that should've redirected
 3. Check out the logs in the Umbraco backoffice
 4. Find the logs related to the request in step 2
 5. Check out what the URL Tracker has done in the logs
The URL Tracker logs every step of the process, including which matchers found a match and how many they've found.
You should first check if your URL is not excluded by any filters. If you find a line in the log that says: `Handling aborted: {reason}`, then your url has been rejected by a filter and you should reevaluate your filter configurations.

If the url has not been rejected, use these steps to find more information about how the URL Tracker attempts to match your incoming url:
 - If you configured a redirect from a static url, check out the static url matcher logs
 - If you configured a redirect from a regular expression, check out the regular expression url matcher logs
 - Always check out the log that says: `No longer available parameters: culture: {culture}, rootnodeid: {rootnodeid}, urls: {urls}`, it will tell you which parameters the URL Tracker has used for matching and you should make sure they match your expectations.

If the matching step is succesful and you see in the logs that a match was found, then something might be up with the handler of the redirect.
If you find a log that says: `Last chance handler invoked for intercept of type {interceptType}. Did you forget to register a handler?` Then you are missing a handler for your custom matcher. Make sure to implement a handler that can handle an intercept of the type given in the log.

<a href="https://infocaster.net">
<img align="right" height="200" src="https://github.com/Infocaster/.github/blob/cba580027a6761844ddab7267f85debe31e96f1a/assets/Infocaster_Corner.png?raw=true">
</a>