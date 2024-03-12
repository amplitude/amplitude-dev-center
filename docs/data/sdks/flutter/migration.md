---
title: Flutter SDK Migration Guide
description: Use this guide to easily migrate from Amplitude's Flutter SDK 3.0 to the Flutter SDK 4.0.
---

Amplitude Flutter SDK 4.0 features default event tracking, simplified interfaces and wraps on the latest [Amplitude iOS SDK](/data/sdks/ios-swift/) and [Android Kotlin SDK](/data/sdks/android-kotlin/). 

!!!beta "SDK Resources"
     [:material-github: GitHub](https://github.com/amplitude/Amplitude-Flutter/tree/beta) · [:material-code-tags-check: Releases](https://github.com/amplitude/experiment-php-server/releases)

## Terminology

* `amplitude_flutter:v3`: Flutter SDK 3.0
* `amplitude_flutter:v4`: Flutter SDK 4.0

## Dependencies

Go to the pubspec.yaml file and update the dependency.

```diff
dependencies:
-  amplitude_flutter: ^3
+  amplitude_flutter: ^4
```

## Instrumentation

Flutter SDK 4.0 offers an API to instrument events. To migrate to the Flutter SDK 4.0, you need to update a few calls. The following sections detail which calls have changed.

### Initialization

Like all other calls, `instance()` has been removed. Configuration is handled differently. The Flutter SDK 4.0 use the Configuration object to set the configuration. See [Configuration](#configuration).

```diff
- import 'package:amplitude_flutter/amplitude.dart';
- import 'package:amplitude_flutter/identify.dart';
+ import 'package:amplitude_flutter/amplitude.dart';
+ import 'package:amplitude_flutter/configuration.dart';

// Create the instance and initialize SDK
- final Amplitude amplitude = Amplitude.getInstance(instanceName: "project");
- amplitude.setServerUrl("https://your.endpoint.com")
- amplitude.setServerZone("US")
- amplitude.trackingSessionEvents(true)
- amplitude.init(widget.apiKey);
+ final Amplitude amplitude = Amplitude(Configuration(
+     apiKey: "YOUR-API-KEY",
+     serverUrl: "https://your.endpoint.com",
+     serverZone: ServerZone.eu,
+     defaultTracking: DefaultTrackingOptions(
+        sessions: true
+     )
+   ));
+ await amplitude.isBuilt;
```

### Configuration

The configurations for the new SDK are simpler and more consistent across runtimes. Some configurations are no longer supported. The Flutter SDK 3.0 uses setters for configuration. The configuration of the Flutter SDK 4.0 can be set by an instance variable. Note the Configuration is immutable (cannot be changed) after being passed to Amplitude.

|Flutter SDK 3.0                      |Flutter SDK 4.0                     |
|-------------------------------------|------------------------------------|
| `enableCoppaControl()` or `disableCoppaControl()` | `config.enableCoppaControl`         |
| `setMinTimeBetweenSessionsMillis()` | `config.minTimeBetweenSessionsMillis` |
| `setEventUploadThreshol(d)`         | `config.flushQueueSize`            |
| `setEventUploadPeriodMillis()`      | `config.flushIntervalMillis`       |
| `setServerZone()`                   | `config.serverZone`                |
| `setServerUrl()`                    | `config.serverUrl`                 |
| `setUseDynamicConfig()`             | NOT SUPPORTED                      |
| `setOptOut()`                       | `config.optOut`                    |
| `setOffline()`                      | NOT SUPPORTED                      |
| `trackingSessionEvents()`           | `config.defaultTracking.sessions`  |
| `useAppSetIdForDeviceId()`          | `config.useAppSetIdForDeviceId`    |

### Tracking events

The Flutter SDK 3.0 offered a variety of `logEvent` APIs with `withEventProperties`, `withApiProperties`, `withUserProperties`, `withGroup`, `withGroupProperties`, `withTimestamp`, `outOfSession`, to override specific properties in the event payload. Amplitude has simplified all these variations into a unified `track` API.

#### `logEvent()`

The `logEvent()` API maps to `track()`.

```diff
- amplitude.logEvent('BUTTON_CLICKED');
+ amplitude.track(event: BaseEvent(eventType:'BUTTON_CLICKED'))
```

#### `logEvent()` with event properties

```diff
- amplitude.logEvent('BUTTON_CLICKED', {"Hover Time": "100ms"});
+ amplitude.track(event: BaseEvent(eventType:'BUTTON_CLICKED', eventProperties: {"Hover Time": "100ms"}))
```

#### `logEvent()` with outOfSession

`logEvent()` receives an optional boolean argument `outOfSession`. The new `track()` API will **NOT** support it. However, you can still track an event as out-of-session by setting `event.sessionId = -1`.

```diff
- amplitude.logEvent("BUTTON_CLICKED", outOfSession: true)
+ amplitude.track(event: BaseEvent(eventType:'BUTTON_CLICKED', sessionId: -1))
```

#### `uploadEvents()`

The `uploadEvents()` API maps to `flush()`.

```diff
- amplitude.uploadEvents();
+ amplitude.flush()
```

### Set user properties

The APIs for setting user properties are the same, except for the removal of `instance()`. Here are code snippets to migrate APIs for user properties.

#### `setUserId()`

`setUserId()` remains unchanged but does **NOT** receive `startNewSession`.

#### `setDeviceId()`

`setDeviceId()` remains unchanged.

#### `regenerateDeviceId()`

`regenerateDeviceId()` is not supported in the Flutter SDK 4.0.

#### `clearUserProperties()`

The `clearUserProperties` API has been removed, but you can now use the unified `identify` API to remove user properties. 

```diff
- amplitude.clearUserProperties();
+ final Identify identify = Identify()
+   ..clearAll()
+ amplitude.identify(identify)
```

#### `setUserProperties()`

The `setUserProperties` API has been removed, but you can now use the unified `identify` API to add user properties. 

```diff
- Map<String, dynamic> userProps = {
-   'gender': 'female',
-   'age': '20'
- };
- amplitude.setUserProperties(userProperties);
+ final Identify identify = Identify()
+                          ..set('gender','female')
+                          ..set('age',20);
+ amplitude.identify(identify);
```

#### `identify()`

You can now make an identify call on `amplitude` without calling `instance()`.

```diff
final Identify identify = Identify()
                          ..set('gender','female')
                          ..set('age',20);
- Amplitude.getInstance().identify(identify);
+ amplitude.identify(identify);
```

### Set group properties

#### `setGroup()`

You can now make an identify call on `amplitude` without calling `instance()`.

```diff
// set group with a single group name
- Amplitude.getInstance().setGroup("orgId", "15");
+ amplitude.setGroup("orgId", "15");
// set group with multiple group names
- Amplitude.getInstance().setGroup("sport", ["tennis", "soccer"]);
+ amplitude.setGroup("sport", ["tennis", "soccer"]);
```

#### `groupIdentify()`

You can now make an identify call on `amplitude` without calling `instance()`.

```diff
final Identify identify = Identify()
                          ..set("gender", "female")
                          ..set("age", 20);
- Amplitude.getInstance().groupIdentify("groupType", "groupValue", identify);
+ amplitude.groupIdentify("groupType", "groupValue", identify);
```

### Tracking revenue

`logRevenue()` and `logRevenueAmount()` are replaced by `revenue()`.

```diff
- String productId = "product001";
- int quantity = 2;
- double price = 20;
- double amount = 35;
- amplitude.logRevenue(productId, quantity, price);
- amplitude.logRevenueAmount(amount);
+ final Revenue revenue = Revenue()
+   ..price = 3.99
+   ..quantity = 3
+   ..productId = "com.company.productId";
+ amplitude.revenue(revenue);
```
