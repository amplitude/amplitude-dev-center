---
title: Session Replay Browser SDK Plugin
---

!!!note "Session Replay Insrumentation"
    Session Replay isn't enabled by default, and requires setup beyond the standard Amplitude instrumentation.

This article covers the installation of Session Replay using the Browser SDK plugin. If your site is already instrumented with Amplitude, use this option. If you use a provider other than Amplitude for in-product analytics, choose the [standalone implementation](/session-replay/sdks/standalone).

--8<-- "includes/session-replay/performance.md"

## Before you begin

Use the latest version of the Session Replay Plugin above version @{$ browser.session_replay.plugin.version $}. For more information, see the [change log](https://github.com/amplitude/Amplitude-TypeScript/blob/v1.x/packages/plugin-session-replay-browser/CHANGELOG.md) on GitHub.

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

You can also add the code directly to the `<head>` of your site. With this method, be sure that the Browser SDK isn't initialized elsewhere in your application. If you initialize the Browser SDK more than once, you may see mismatches in Device ID or Session ID.

```html
<script src="https://cdn.amplitude.com/libs/analytics-browser-2.1.3-min.js.gz"></script>
<script src="https://cdn.amplitude.com/libs/plugin-session-replay-browser-@{$ browser.session_replay.plugin.version $}-min.js.gz"></script>
<script>
window.amplitude.init(API_KEY)
const sessionReplayTracking = window.sessionReplay.plugin();
window.amplitude.add(sessionReplayTracking);
</script>
```

!!! tip "Compatability with Google Tag Manager"
    The Session Replay plugin scripts load asynchronously when you add them to the `<head>` tag of your page. As a result, this implementation isn't compatible with Google Tag Manager. For more information, see [Session Replay Implementation with Google Tag Manager](/session-replay/tag-managers/google-tag-manager).

## Configuration

Pass the following option when you initialize the Session Replay plugin:

| Name              | Type      | Required | Default         | Description                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ----------------- | --------- | -------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sampleRate`      | `number`  | No       | `0`             | Use this option to control how many sessions to select for replay collection. <br></br>The number should be a decimal between 0 and 1, for example `0.4`, representing the fraction of sessions to have randomly selected for replay collection. Over a large number of sessions, `0.4` would select `40%` of those sessions. |

### Track default session events

Session Replay enables session tracking by default. This ensures that Session Replay captures Session Start and Session End events. If you didn't capture these events before you implement Session Replay, expect an increase in event volume. For more information about session tracking, see [Browser SDK 2.0 | Tracking Sessions](/data/sdks/browser-2/#tracking-sessions).

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

!!! info "Session Start and Session End events"
    Beginning with plugin version 1.1.0, Session Replay no longer requires `Session Start` and `Session End` events, but does capture them by default. To disable capture of these events, set `forceSessionTracking: false`:

    ```js
    const sessionReplayTracking = window.sessionReplay.plugin({ 
        forceSessionTracking: false, 
        sampleRate: 1, // 100% sample rate, should reduce for production traffic.  
    }); 
    ```

    Amplitude requires at least one event in any captured session to enable playback of the replay. Amplitude recommends that you leave Session Tracking enabled, as that triggers `[Amplitude] Session Start` at the beginning of the session. If you disable session tracking, configure another event to fire early in the session for best results.
  
--8<-- "includes/session-replay/mask-onscreen-data.md"

### User opt-out

The Session Replay plugin follows the Browser SDK's `optOut` setting, and doesn't support user opt-outs on its own.

--8<-- "includes/session-replay/eu-data-residency.md"

--8<-- "includes/session-replay/sampling-rate.md"

### Disable replay collection

Once enabled, Session Replay runs on your site until either:

- The user leaves your site
- You call `amplitude.remove('sessionReplayTracking')`

Call `amplitude.remove('sessionReplayTracking')` before a user navigates to a restricted area of your site to disable replay collection while the user is in that area. 

To restart replay collection, call `amplitude.add('sessionReplayTracking')` to re-add the plugin.

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

For more information about individual statuses and errors, see the [Session Replay Ingestion Monitor](/session-replay/ingestion-monitor).

### CSS styling doesn't appear in replay

When Amplitude captures a replay, it doesn't download and store CSS files or other static assets that are part of your application or site. Session Replay stores references to these files, and uses those references while it reconstructs the replay. In some situations, the styling present in the replay may differ from your application for the following reasons:

- Assets on your site move or change name. This can happen when you deploy a new version of your application.
- Assets on your site are behind access controls that prevent Amplitude from fetching them.

To help resolve CSS loading issues:

- Ensure your domain is publicly accessible. If you store assets on `localhost`, try moving them to a staging environment.
- Your CDN should keep track of old stylesheets for older replays. If the content of the same stylesheet changes over time, try to append a unique string or hash to the asset URL. For example, `stylesheet.css?93f8b89`.
- Add `app.amplitude.com` or `app.eu.amplitude.com` to the list of domains that your server's CORS configuration permits.

### Captured sessions contain limited information

The Session Replay Plugin enables session tracking by default. If you instrument events outside of the Browser SDK, Amplitude doesn't tag those events as part of the session replay. This means you can't use tools like Funnel, Segmentation, or Journeys charts to find session replays. You can find session replays with the User Sessions chart or through User Lookup.

If you use a method other than the Browser SDK to instrument your events, consider using the [Session Replay Standalone SDK](/session-replay/sdks/standalone/).

### Replay length and session length don't match

In some scenarios, the length of a replay may exceed the time between the `[Amplitude] Start Session` and `[Amplitude] End Session` events. This happens when a user closes the `[Amplitude] End Session` occurs, but before the Browser SDK and Session Replay plugin can process it. When the user visits that page again, the SDK and plugin process the event and send it to Amplitude, along with the replay. You can verify this scenario occurs if you see a discrepancy between the `End Session Client Event Time` and the `Client Upload Time`.

### Session replays don't appear in Amplitude 

Session replays may not appear in Amplitude due to:

- Content security policy
- Blocked JavaScript
- No events triggered through the browser SDK in the current session
- Sampling

#### Local development and focus state

The Session Replay SDK and plugin capture only the page that's in focus. When you develop locally with the browser console open, focus states may not work as expected. If you don't see replays in Amplitude, try to enable `debugMode`. In this mode, Session Replay ignores the focus handle and enables extra debugging information.

```js title="Enable debug mode"
const sessionReplayTracking = window.sessionReplay.plugin({
        debugMode: true, 
        sampleRate: 1, 
    });
```

#### Content security policy

When you add the Session Replay script to your site, visit a page on which the Session Replay SDK is running, and open your browser's developer tools.

Check for any error messages in the JavaScript console that contain the text `Content Security Policy`. For example, `Refused to connect to 'https://api-secure.amplitude.com/sessions/track' because it violates the document's Content Security Policy`.

To resolve this error, update your site's content security policy to allow connection to Amplitude's APIs.

#### Blocked JavaScript

Browser extensions or network security policy may block the Session Replay SDK. Check your browser's developer tools to see if requests fail, and if so, add an exception for the blocked domains.

#### No events triggered through the browser SDK in the current session

Session Replay requires that at least one event in the user's session has the `[Amplitude] Session Replay ID` property. The [Browser SDK](/data/sdks/browser-2/) `Session Start` and `Session End` events include this property by default. If you instrument your events with any SDK other than the Amplitude Browser SDK 2, use the [Session Replay Standalone SDK](/session-replay/sdks/standalone) and ensure you tag your events with the necessary event properties.

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
- Replays older than the set [retention period](#retention-period) (defaults to 30 days, or 90 days if you purchase extra volume).
