---
title: Session Replay Standalone SDK for Browser
---

!!!note "Session Replay Insrumentation"
    Session Replay isn't enabled by default, and requires setup beyond the standard Amplitude instrumentation.
 
This article covers the installation of Session Replay using the standalone SDK. If you use a provider other than Amplitude for in-product analytics, choose this option. If your site is already instrumented with Amplitude Browser SDK, use the [Session Replay Browser SDK Plugin](/session-replay/sdks/plugin).

--8<-- "includes/session-replay/performance.md"

## Before you begin

Use the latest version of the Session Replay standalone SDK above version 1.0.1. For more information, see the [change log](https://github.com/amplitude/Amplitude-TypeScript/blob/v1.x/packages/session-replay-browser/CHANGELOG.md) on GitHub.

Session Replay Standalone SDK requires that:

1. Your application is web-based.
2. You track sessions with a timestamp, which you can pass to the SDK. You inform the SDK whenever a session timestamp changes.
3. You can provide a device ID to the SDK.
4. The `Session ID` and `Device ID` you pass to the Standalone SDK must match those sent as event properties to Amplitude.

The Standalone SDK doesn't provide Session management capabilities. Your application or a third-party integration must update the SDK with changes to `Session ID` and `Device ID`. 

--8<-- "includes/session-replay/browsers.md"

## Quickstart

Install the standalone SDK with yarn or npm.

=== "npm"

    ```bash
    npm install @amplitude/session-replay-browser --save
    ```

=== "yarn"

    ```bash
    yarn add @amplitude/session-replay-browser
    ```

Configure your application code.

1. Call `sessionReplay.init` to begin collecting replays. Pass the API key, session identifier, and device identifier.
2. When the session identifier changes, pass the new value to Amplitude with `sessionReplay.setSessionId`.
3. Collect Session Replay properties to send with other event properties with `sessionReplay.getSessionReplayProperties`. See [Add Session Replay ID to your events](#add-session-replay-id-to-your-events) for more information.

```javascript
import * as sessionReplay from "@amplitude/session-replay-browser";
import 3rdPartyAnalytics from 'example'

const AMPLITUDE_API_KEY = "key"

// Configure the SDK and begin collecting replays
await sessionReplay.init(AMPLITUDE_API_KEY, {
  deviceId: "<string>",
  sessionId: "<number>",
  optOut: "<boolean>",
  sampleRate: "<number>"
}).promise;

// Call whenever the session id changes
sessionReplay.setSessionId(sessionId);

// When you send events to Amplitude, call this event to get
// the most up to date session replay properties for the event
const sessionReplayProperties = sessionReplay.getSessionReplayProperties();
3rdPartyAnalytics.track('event', {...eventProperties, ...sessionReplayProperties})
```

## Add Session Replay ID to your events

The Session Replay SDK outputs the Session Replay properties that you need to add to your custom event instrumentation. `getSessionReplayProperties` returns event properties, namely the `[Amplitude] Session Replay ID` event property that you need to add to events before you send them to Amplitude. An example response of getSessionReplayProperties is: 

```js
{
    "[Amplitude] Session Replay ID": "6eb24f81-a106-45b0-879c-65248d7b8911/1710374872575"
}
```

!!!info
    `getSessionReplayProperties` may return an empty object if Session Replay doesn't capture the session (for example, due to sampling or if the page is out of focus).

`[Amplitude] Session Replay ID` is a unique identifier for the replay, and is different from `[Amplitude] Session ID`, which is the identifier for the user's session.

The [Session Replay Browser Plugin](/session-replay/sdks/plugin) handles this by default, since Amplitude manages event instrumentation. With the Standalone SDK, you need to instrument your application to add this property to any events that occur during capture. 

## Configuration

Pass the following configuration options when you initialize the Session Replay SDK.

| Name              | Type      | Required | Default         | Description                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ----------------- | --------- | -------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `deviceId`        | `string`  | Yes      | `undefined`     | Sets an identifier for the device running your application.                                                                                                                                                                                                                                                                                                                                                                                 |
| `sessionId`       | `number`  | Yes      | `undefined`     | Sets an identifier for the users current session. The value must be in milliseconds since epoch (Unix Timestamp).                                                                                                                                                                                                                                                                                                                           |
| `sampleRate`      | `number`  | No       | `0`             | Use this option to control how many sessions to select for replay collection. <br></br>The number should be a decimal between 0 and 1, for example `0.4`, representing the fraction of sessions to have randomly selected for replay collection. Over a large number of sessions, `0.4` would select `40%` of those sessions. |
| `optOut`          | `boolean` | No       | `false`         | Sets permission to collect replays for sessions. Setting a value of true prevents Amplitude from collecting session replays.                                                                                                                                                                                                                                                                                                                |
| `flushMaxRetries` | `number`  | No       | `5`             | Sets the maximum number of retries for failed upload attempts. This is only applicable to errors that Amplitude can retry.                                                                                                                                                                                                                                                                                                                                 |
| `logLevel`        | `number`  | No       | `LogLevel.Warn` | `LogLevel.None` or `LogLevel.Error` or `LogLevel.Warn` or `LogLevel.Verbose` or `LogLevel.Debug`. Sets the log level.                                                                                                                                                                                                                                                                                                                       |
| `loggerProvider`  | `Logger`  | No       | `Logger`        | Sets a custom `loggerProvider` class from the Logger to emit log messages to desired destination.                                                                                                                                                                                                                                                                                                                                             |
| `serverZone`      | `string`  | No       | `US`            | EU or US. Sets the Amplitude server zone. Set this to EU for Amplitude projects created in EU data center.                                                                                                                                                                                                                                                                                                                                  |

--8<-- "includes/session-replay/mask-onscreen-data.md"

### User opt-out

Session Replay provides an option for opt-out configuration. This prevents Amplitude from collecting session replays when passed as part of initialization. For example:

```js
// Pass a boolean value to indicate a users opt-out status
await sessionReplay.init(AMPLITUDE_API_KEY, {
  optOut: true,
}).promise;
```

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
import * as sessionReplay from "@amplitude/session-replay-browser";
import 3rdPartyAnalytics from 'example'

const AMPLITUDE_API_KEY = <...>
sessionReplay.init(AMPLITUDE_API_KEY, {
  deviceId: <string>,
  sessionId: <number>,
  optOut: <boolean>,
  sampleRate: <number>
})

if (nonEUCountryFlagEnabled) {
  const sessionReplayProperties = sessionReplay.getSessionReplayProperties();
  3rdPartyAnalytics.track('event', {...eventProperties, ...sessionReplayProperties})
}
```

--8<-- "includes/session-replay/data-retention.md"

--8<-- "includes/session-replay/replay-storage.md"

--8<-- "includes/session-replay/known-limitations.md"

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

### Session replays don't appear in Amplitude 

Session replays may not appear in Amplitude due to:

- Content security policy
- Blocked JavaScript
- Sampling

#### Content security policy

When you add the Session Replay script to your site, visit a page on which the Session Replay SDK is running, and open your browser's developer tools.

Check for any error messages in the JavaScript console that contain the text `Content Security Policy`. For example, `Refused to connect to 'https://api-secure.amplitude.com/sessions/track' because it violates the document's Content Security Policy`.

To resolve this error, update your site's content security policy to allow connection to Amplitude's APIs.

#### Blocked JavaScript

Browser extensions or network security policy may block the Session Replay SDK. Check your browser's developer tools to see if requests fail, and if so, add an exception for the blocked domains.

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

## Use Session Replay with Segment analytics

Session Replay supports other analytics providers. Follow the information below to add Session Replay to an existing Segment-instrumented site.

- [Amplitude (Actions)](#amplitude-actions-destination)
- [Amplitude Classic (Cloud-mode)](#amplitude-classic-destination-cloud-mode)
- [Amplitude Classic (Device-mode)](#amplitude-classic-destination-device-mode)

### Amplitude (Actions) destination

Amplitude (Actions) tracks sessions automatically. When a user starts a new session, Amplitude sets an `analytics_session_id` cookie on the users browser. Configure your implementation to listen for changes in value to `analytics_session_id`, which you can do with Segment's [Source Middleware](https://segment.com/docs/connections/sources/catalog/libraries/website/javascript/middleware/).

This code snippet shows how to configure Session Replay with Segment's Amplitude (Actions) integration, and update the session ID when `analytics_session_id` changes.

```javascript
import * as sessionReplay from "@amplitude/session-replay-browser";
import { AnalyticsBrowser } from "@segment/analytics-next";

const segmentAnalytics = AnalyticsBrowser.load({
  writeKey: "segment-key",
});

const AMPLITUDE_API_KEY = 'api-key' // must match that saved with Segment

const getStoredSessionId = () => {
 return cookie.get("amp_session_id") || 0;
}

const user = await segmentAnalytics.user();
const storedSessionId = getStoredSessionId();
await sessionReplay.init(AMPLITUDE_API_KEY, { 
  sessionId: storedSessionId,
  deviceId: user.anonymousId()
}).promise;

// Add middleware to check if the session id has changed, 
// and update the session replay instance
segmentAnalytics.addSourceMiddleware(({ payload, next, integrations }) => {
  const storedSessionId = getStoredSessionId();
  const nextSessionId = payload.obj.integrations['Actions Amplitude'].session_id || 0

  if (storedSessionId < nextSessionId) {
    cookie.set("amp_session_id", nextSessionId);
    sessionReplay.setSessionId(nextSessionId);
  }
  next(payload);
});

// Add middleware to always add session replay properties to track calls
SegmentAnalytics.addSourceMiddleware(({ payload, next, integrations }) => {
  const sessionReplayProperties = sessionReplay.getSessionReplayProperties();
  if (payload.type() === "track") {
    payload.obj.properties = {
      ...payload.obj.properties,
      ...sessionReplayProperties,
    };
  }
  
  next(payload);
});
```

### Amplitude Classic destination (Device-mode)

This version of the Amplitude destination installs the Amplitude JavaScript SDK (5.2.2) on the client, and sends events directly to `api.amplitude.com`.

The Device-mode integration tracks sessions by default, since it includes the amplitude-js SDK. The included SDK version (5.2.2) doesn't include an event for session changes. As a result, use Segment's middleware to update Session Replay when the session ID changes.

```javascript
import * as sessionReplay from "@amplitude/session-replay-browser";
import { AnalyticsBrowser } from "@segment/analytics-next";

const segmentAnalytics = AnalyticsBrowser.load({
  writeKey: "segment-key",
});

const AMPLITUDE_API_KEY = 'api-key' // must match that saved with Segment

const getAmpSessionId = () => {
  const sessionId = window.amplitude.getInstance().getSessionId();
  cookie.set("amp_session_id", sessionId);
  return sessionId;
};

// Wait for the amplitude-js SDK to initialize,
// then initialize session replay with the correct device id and session id
window.amplitude.getInstance().onInit(() => {
  const sessionId = getAmpSessionId();
  sessionReplay.init(AMPLITUDE_API_KEY, {
    deviceId: window.amplitude.getInstance().options.deviceId,
    sessionId: getAmpSessionId(),
  });
});


// Add middleware to check if the session id has changed, 
// and update the session replay instance
SegmentAnalytics.addSourceMiddleware(({ payload, next, integrations }) => {
  const nextSessionId = window.amplitude.getInstance().getSessionId();
  const storedSessionId = cookie.get("amp_session_id") || 0;

  if (storedSessionId < nextSessionId) {
    cookie.set("amp_session_id", nextSessionId);
    sessionReplay.setSessionId(nextSessionId);
  }
  next(payload);
});

// Add middleware to always add session replay properties to track calls
SegmentAnalytics.addSourceMiddleware(({ payload, next, integrations }) => {
  const sessionReplayProperties = sessionReplay.getSessionReplayProperties();
  if (payload.type() === "track") {
    payload.obj.properties = {
      ...payload.obj.properties,
      ...sessionReplayProperties,
    };
  }
  
  next(payload);
});
```

### Amplitude Classic destination (Cloud-mode)

This version of the Amplitude destination sends events to Segment's backend, which forwards them to Amplitude. The Cloud-mode destination doesn't track sessions by default. To overcome this, use the Browser SDK as a shell to manage sessions, and use Session Replay as a plugin, as shown below.

```javascript
import { sessionReplayPlugin } from "@amplitude/plugin-session-replay-browser";
import { AnalyticsBrowser } from "@segment/analytics-next";

const segmentAnalytics = AnalyticsBrowser.load({
  writeKey: "segment-key",
});

// A plugin must be added so that events sent through Segment will have
// session replay properties and the correct session id
const segmentPlugin = () => {
  return {
    name: "segment",
    type: "destination",
    execute: async (event) => {
      const properties = event.event_properties || {};
      segmentAnalytics.track(event.event_type, properties, {
        integrations: {
          Amplitude: {
            session_id: amplitude.getSessionId(),
          },
        },
      });
      return {
        code: 200,
        event: event,
        message: "OK",
      };
    },
  };
}; 

const AMPLITUDE_API_KEY = 'api-key' // must match that saved with Segment

// Add the session replay plugin first, then the segment plugin
await amplitude.add(sessionReplayPlugin()).promise;
await amplitude.add(segmentPlugin()).promise;

const user = await segmentAnalytics.user();
await amplitude.init(AMPLITUDE_API_KEY, { 
  instanceName: 'session-replay',
  sessionTimeout: Number.MAX_SAFE_INTEGER,
  defaultTracking: false,
  deviceId: user.anonymousId()
}).promise;
amplitude.remove('amplitude');

// Events must be tracked through the shell Browser SDK to properly attach
// session replay properties
amplitude.track('event name')
```
