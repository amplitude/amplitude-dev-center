#### How to debug

Debugging in a browser can help you identify problems related to your code's implementation, as well as potential issues within the SDKs you're using. Here's a basic guide on how to use the browser's built-in Developer Tools (DevTools) for debugging.

##### Console

You can find JavaScript errors under **Inspect > Console**, which might have the details about the line of code and file that caused the problem. The console also allows you to execute JavaScript code in real time.

* Enable debug mode by following these [instructions](./#debug-mode). Then With the default logger, extra function context information will be output to the developer console when any SDK public method is invoked, which can be helpful for debugging.

* Amplitude supports SDK deferred initialization. Events tracked before initialization will be dispatched after the initialization call. If you cannot send events but can send the event successfully after entering `amplitude.init(API_KEY, 'USER_ID')` in the browser console, it indicates that your `amplitude.init` call might not have been triggered in your codebase or you are not using the correct amplitude instance during initialization. Therefore, please check your implementation."

##### Network Request

Use the **Inspect > Network** tab to view all network requests made by your page. Search for the Amplitude request.
![sdk debugging network request](../../../assets/images/sdk-debuggability-network-request.png)

Please check the response code and ensure that the response payload is as expected.

##### Instrumentation Explorer/Chrome Extension

The Amplitude Instrumentation Explorer is an extension available in the Google Chrome Web Store. The extension captures each Amplitude event you trigger and displays it in the extension popup. It's important to ensure that the event has been sent out successfully and to check the context in the event payload.

Check [here](../../debugger/#step-2-analyze-the-event-stream) for more details.

#### Common Issues

The following are common issues specific to Browser SDK. For additional general common issues, please refer to [this document](../../sdk-troubleshooting-and-debugging/).

##### AD Blocker

`Ad Blocker` might lead to event dropping. The following errors indicate that the tracking has been affected by `Ad Blocker`. When loading via a script tag, an error may appear in the console/network tab while loading the SDK script. When loaded with npm package, there could be errors in the network tab when trying to send events to the server. The errors might vary depending on the browser.

* Chrome (Ubuntu, MacOS)
Console: error net::ERR_BLOCKED_BY_CLIENT
Network: status (blocked:other)
* Firefox (Ubuntu)
Console: error text doesn’t contain any blocking-specific info
Network: Transferred column contains the name of plugin Blocked by uBlock Origin
* Safari (MacOS)
Console: error contains text Content Blocker prevented frame ... from loading a resource from ...
Network: it looks like blocked requests are not listed. Not sure if it’s possible to show them.

We recommend using a proxy server to avoid this situation.

##### Cookies related

Here is the [information](./#cookie-management) SDK stored in the cookies. This means that client behavior, like disabling cookies or using a private browser/window/tab, will affect the persistence of these saved values in the cookies. So, if these values are not persistent or are not increasing by one, that could be the reason.

##### CORS

Cross-Origin Resource Sharing (CORS) is a security measure implemented by browsers to restrict how resources on a web page can be requested from a different domain. It might cause this issue if you used `setServerURL`.

```Access to fetch at 'xxx' from origin 'xxx' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.```

Cross-origin resource sharing (CORS) prevents a malicious site from reading another site's data without permission. The error message suggests that the server you're trying to access is not allowing your origin to access the requested resource. This is due to the lack of the `Access-Control-Allow-Origin` header in the server's response.

* If you have control over the server, you can "Update the server's CORS policy". Add the `Access-Control-Allow-Origin` header to the server's responses. This would allow your origin to make requests. The value of `Access-Control-Allow-Origin` can be * to allow all origins, or it can be the specific URL of your web page.

* If you don't have control over the server, you can set up a proxy server that adds the necessary CORS headers. The web page makes requests to the proxy, which then makes requests to the actual server. The proxy adds the `Access-Control-Allow-Origin` header to the response before sending it back to the web page.

If you have set up an API proxy and run into configuration issues related to that on a platform you’ve selected, that’s no longer an SDK issue but an integration issue between your application and the service provider.

##### Events fired but no network requests

If you [set the logger to "Debug" level](./#debug-mode), and see track calls in the developer console, the `track()` method has been called. If you don't see the corresponding event in Amplitude, the Amplitude Instrumentation Explorer Chrome extension, or the network request tab of the browser, the event wasn't sent to Amplitude. Events are fired and placed in the SDK's internal queue upon a successful `track()` call, but sometimes these queued events may not send successfully. This can happen when an in-progress HTTP request is cancelled. For example, if you close the browser or leave the page.

There are two ways to address this issue:

1. If you use standard network requests, set the transport to `beacon` during initialization or set the transport to `beacon` upon page exit. `sendBeacon` doesn't work in this case because it sends events in the background, and doesn't return server responses like `4xx` or `5xx`. As a result, it doesn't retry on failure. `sendBeacon` sends only scheduled requests in the background. For more information, see the [sendBeacon](./#use-sendbeacon) section.

2. To make track() synchronous, [add the `await` keyword](./#callback) before the call.


