---
title: Session Replay Browser SDK Plugin
---

This article covers the installation of Session Replay using the Browser SDK plugin. If your site is already instrumented with Amplitude, use this option. If you use a provider other than Amplitude for in-product analytics, choose the [standalone implementation](/session-replay/sdks/standalone).

--8<-- "includes/session-replay/performance.md"

## Before you begin

Use the latest version of the Session Replay Plugin above version 0.8.1. For more information, see the [change log](https://github.com/amplitude/Amplitude-TypeScript/blob/v1.x/packages/plugin-session-replay-browser/CHANGELOG.md) on GitHub.

The Session Replay Plugin requires that:

1. Your application is web-based.
2. You can provide a device ID to the SDK.

--8<-- "includes/session-replay/browsers.md"

## Quickstart

Install the plugin with yarn or npm.

=== "npm"

    ```bash
    npm install @amplitude/plugin-session-replay-browser --save
    ```

=== "yarn"

    ```bash
    yarn add @amplitude/plugin-session-replay-browser
    ```

Configure your application code.

```js
import * as amplitude from '@amplitude/analytics-browser';
import { sessionReplayPlugin } from '@amplitude/plugin-session-replay-browser';

// Your existing initialization logic with Browser SDK
amplitude.init(API_KEY);

// Create and Install Session Replay Plugin
const sessionReplayTracking = sessionReplayPlugin();
amplitude.add(sessionReplayTracking);
```

You can also add the code directly to the `<head>` of your site.

```html
<script src="https://cdn.amplitude.com/libs/analytics-browser-2.1.3-min.js.gz"></script>
<script src="https://cdn.amplitude.com/libs/plugin-session-replay-browser-0.10.1-min.js.gz"></script>
<script>
window.amplitude.init(API_KEY)
const sessionReplayTracking = window.sessionReplay.plugin();
window.amplitude.add(sessionReplayTracking);
</script>
```

## Configuration

Pass the following option when you initialize the Session Replay plugin:

| Name              | Type      | Required | Default         | Description                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ----------------- | --------- | -------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sampleRate`      | `number`  | No       | `0`             | Use this option to control how many sessions to select for replay collection. <br></br>The number should be a decimal between 0 and 1, for example `0.4`, representing the fraction of sessions to have randomly selected for replay collection. Over a large number of sessions, `0.4` would select `40%` of those sessions. |
| `logLevel`        | `number`  | No       | `LogLevel.Warn` | `LogLevel.None` or `LogLevel.Error` or `LogLevel.Warn` or `LogLevel.Verbose` or `LogLevel.Debug`. Sets the log level.                                                                                                                                                                                                                                                                                                                       |

### Track default session events

Session replay requires that you configure default session event tracking. This ensures that Session Replay captures Session Start and Session End events. If you didn't capture these events before you implement Session Replay, expect an increase in event volume. For more information about session tracking, see [Browser SDK 2.0 | Tracking Sessions](/data/sdks/browser-2/#tracking-sessions).

=== "SDK Configuration"

    Use the Browser SDK configuration to implicitly enable session tracking.

    ```js 
    amplitude.init(API_KEY, USER, {
        defaultTracking: {
            sessions: true
        }
    });
    ```

=== "Plugin Configuration"

    Disable all default tracking by the Browser SDK. In this case, the plugin enables default session tracking.

    ```js
    amplitude.init(API_KEY, USER, {
        defaultTracking: false
    });
    ```

--8<-- "includes/session-replay/mask-onscreen-data.md"

--8<-- "includes/session-replay/user-opt-out.md"

--8<-- "includes/session-replay/eu-data-residency.md"

--8<-- "includes/session-replay/sampling-rate.md"

### Disable replay collection

Once enabled, Session Replay runs on your site until either:

- The user leaves your site
- You call `sessionReplay.shutdown()`

Call `sessionReplay.shutdown()` before a user navigates to a restricted area of your site to disable replay collection while the user is in that area. 

Call `sessionReplay.init(API_KEY, {...options})` to re-enable replay collection when the return to an unrestricted area of your site.

You can also use a feature flag product like Amplitude Experiment to create logic that enables or disables replay collection based on criteria like location. For example, you can create a feature flag that targets a specific user group, and add that to your initialization logic:

```javascript
import { sessionReplayPlugin } from "@amplitude/plugin-session-replay-browser";

// Your existing initialization logic with Browser SDK
amplitude.init(API_KEY);

if (nonEUCountryFlagEnabled) {
  // Create and Install Session Replay Plugin
  const sessionReplayTracking = sessionReplayPlugin({
    sampleRate: 0.5,
  });
  amplitude.add(sessionReplayTracking);
}
```

--8<-- "includes/session-replay/data-retention.md"

--8<-- "includes/session-replay/replay-storage.md"

--8<-- "includes/session-replay/known-limitations.md"

### Multiple Amplitude instances

Session Replay supports attaching to a single instance of the Amplitude SDK. If you have more than one instance instrumented in your application, make sure to start Session Replay on the instance that most relates to your project.

```html
<script>
  const sessionReplayTracking = window.sessionReplay.plugin();
  const instance = window.amplitude.createInstance();
  instance.add(sessionReplayTracking);
  instance.init(API_KEY);
<script>

```

## Troubleshooting

### Captured sessions contain limited information

Session Replay requires that the Browser SDK send Session Start and Session End events, at a minimum. If you instrument events outside of the Browser SDK, Amplitude doesn't tag those events as part of the session replay. This means you can't use tools like Funnel, Segmentation, or Journeys charts to find session replays. You can find session replays with the User Sessions chart or through User Lookup.

If you use a method other than the Browser SDK to instrument your events, consider using the [Session Replay Standalone SDK](/session-replay/sdks/standalone/).

### Session replays don't appear in Amplitude 

Session replays may not appear in Amplitude due to:

- Content security policy
- Blocked JavaScript
- No events triggered through the browser SDK in the current session
- Sampling

#### Content security policy

When you add the Session Replay script to your site, visit a page on which the Session Replay SDK is running, and open your browser's developer tools.

Check for any error messages in the JavaScript console that contain the text `Content Security Policy`. For example, `Refused to connect to 'https://api-secure.amplitude.com/sessions/track' because it violates the document's Content Security Policy`.

To resolve this error, update your site's content security policy to allow connection to Amplitude's APIs.

#### Blocked JavaScript

Browser extensions or network security policy may block the Session Replay SDK. Check your browser's developer tools to see if requests fail, and if so, add an exception for the blocked domains.

#### No events triggered through the browser SDK in the current session

Session Replay requires that at least one event in the user's session has the `[Amplitude] Session Replay ID` property. If you instrument your events with a method other than the [Browser SDK](/data/sdks/browser-2/), the Browser SDK may send only the default Session Start and Session End events, which don't include this property.

For local testing, you can force a Session Start event to ensure that Session Replay functions. 

1. Open your browser's developer tools, and delete any cookie that begins with `AMP_`. 
2. Close developer tools and refresh the page.
3. In Amplitude, in the User Lookup Event Stream, you should see a Session Start event that includes the `[Amplitude] Session Replay ID` property. After processing, the Play Session button should appear for that session.

#### Sampling

As mentioned above, the default `sampleRate` for Session Replay is `0`. Update the rate to a higher number. For more information see, [Sampling rate](#sampling-rate).

#### Some sessions don't include the Session Replay ID property

Session replay doesn't require that all events in a session have the `[Amplitude] Session Replay ID` property, only that one event in the session has it. Reasons why `[Amplitude] Session Replay ID`  may not be present in an event include:

- If you instrument an event with a source different from the source you connect to Session Replay. For example, your application may send events from a backend source, rather than the Browser SDK.
- If events fire when the user isn't focused on the page. Session Replay pauses the SDK when user focus leaves the page. Amplitude events may still send through your provider, but `getSessionReplayProperties()` doesn't return the `[Amplitude] Session Replay ID` property. This is because Session Replay hasn't begun the capture, since the user hasn't interacted with the page. This should lead to a decrease in the amount of inactivity that a replay captures.

### Session Replay processing errors

In general, replays should be available within minutes of ingestion. Delays or errors may be the result of one or more of the following:

- Mismatching API keys or Device IDs. This can happen if Session Replay and standard event instrumentation use different API keys or Device IDs.
- Session Replay references the wrong project.
- Short sessions. If a users bounces within a few seconds of initialization, the SDK may not have time to upload replay data.
- Page instrumentation. If Session Replay isn't implemented on all pages a user visits, their session may not capture properly.
- Replays older than the set [retention period](#retention-period) (defaults to 90 days).

### Session Replay shows warnings in the user's browser console

Session Replay provides a `logLevel` configuration setting to customize the level of error that Session Replay logs to the console. Decrease the level of logging to mitigate JavaScript errors in the console.