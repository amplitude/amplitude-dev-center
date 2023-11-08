---
title: Experiment iOS SDK
description: Official documentation for Amplitude Experiment's Client-side iOS SDK.
icon: simple/ios
---

Official documentation for Amplitude Experiment's Client-side iOS SDK implementation.

!!!info "SDK Resources"
    [:material-github: GitHub](https://github.com/amplitude/experiment-ios-client) · [:material-code-tags-check: Releases](https://github.com/amplitude/experiment-ios-client/releases) · [:material-book: API Reference](https://amplitude.github.io/experiment-ios-client/)

## Install

!!!bug "Import Statement"
    CocoaPods and Swift Package Manager/Carthage have different import statements.

    * CocoaPods: `import AmplitudeExperiment`
    * SPM/Carthage: `import Experiment`

![CocoaPods](https://img.shields.io/cocoapods/v/AmplitudeExperiment)

=== "CocoaPods"

    ```ruby
    pod 'AmplitudeExperiment', '~> <VERSION>'
    ```

=== "Swift package manager"

    * Package URL: `https://github.com/amplitude/experiment-ios-client`

!!!tip "Quick Start"

    The right way to initialize the Experiment SDK depends on whether you use an
    Amplitude SDK for analytics or a third party (for example, Segment).

    === "Amplitude Analytics"

        1. [Initialize the experiment client](#initialize)
        2. [Start the SDK](#start)
        3. [Access a flag's variant](#variant)

        ```swift
        // (1) Initialize the experiment client with Amplitude Analytics
        let experiment = Experiment.initializeWithAmplitudeAnalytics(
            apiKey: "DEPLOYMENT_KEY",
            config: ExperimentConfigBuilder().build()
        )

        // (2) Start the SDK
        experiment.start() { error in

            // (3) Lookup a flag's variant
            let variant = experiment.variant("FLAG_KEY")
            if variant.value == "on" {
                // Flag is on
            } else {
                // Flag is off
            }
        }
        ```

    === "Third Party Analytics"

        1. [Initialize the experiment client](#initialize)
        2. [Start the SDK with a user](#fetch)
        3. [Access a flag's variant](#variant)

        ```swift

        // (1) Initialize the experiment client and implement a
        //     custom exposure tracking provider.
        class ExposureTracker: ExposureTrackingProvider {
            func track(exposure: Exposure) {
                // TODO: Implement exposure tracking
                // analytics.track(name: "$exposure", properties: [
                //     "flag_key": exposure.flagKey,
                //     "variant": exposure.variant,
                //     "experiment_key": exposure.experimentKey
                // ])
            }
        }
        let experiment = Experiment.initialize(
            apiKey: "DEPLOYMENT_KEY",
            config: ExperimentConfigBuilder()
                .exposureTrackingProvider(ExposureTracker())
                .build()
        )
        // (2) Start the SDK with the user.
        let user = ExperimentUserBuilder()
            .userId("user@company.com")
            .deviceId("abcdefg")
            .userProperty("premium", value: true)
            .build()
        experiment.start(user) { error in

            // (3) Lookup a flag's variant
            let variant = experiment.variant("FLAG_KEY")
            if variant.value == "on" {
                // Flag is on
            } else {
                // Flag is off
            }
        }
        ```

    **Not getting the expected variant result for your flag?** Make sure your flag [is activated](../guides/getting-started/create-a-flag.md#activate-the-flag), has a [deployment set](../guides/getting-started/create-a-flag.md#add-a-deployment), and has [users allocated](../guides/getting-started/create-a-flag.md#configure-targeting-rules).

## Core functions

The following functions make up the core of the Experiment client-side SDK.

---

### Initialize

The SDK client should be initialized in your application on startup. The [deployment key](../general/data-model.md#deployments) argument passed into the `apiKey` parameter must live within the same project that you are sending analytics events to.

=== "Amplitude Analytics"

    ```swift
    func initializeWithAmplitudeAnalytics(
        apiKey: String,
        config: ExperimentConfig
    ) -> ExperimentClient
    ```

=== "Third Party Analytics"

    ```swift
    func initialize(apiKey: String, config: ExperimentConfig) -> ExperimentClient
    ```

| <div class='med-column'>Parameter</div> | Requirement | Description |
| --- | --- | --- |
| `apiKey` | required | The [deployment key](../general/data-model.md#deployments) which authorizes fetch requests and determines which flags should be evaluated for the user. |
| `config` | optional | The client [configuration](#configuration) used to customize SDK client behavior. |

The initializer returns a singleton instance, so subsequent initializations for the same instance name will always return the initial instance. To create multiple instances, use the `instanceName` [configuration](#configuration).

=== "Amplitude Analytics"

    ```swift
    let experiment = Experiment.initializeWithAmplitudeAnalytics(
        apiKey: "DEPLOYMENT_KEY",
        config: ExperimentConfigBuilder().build()
    )
    ```

=== "Third Party Analytics"

    ```swift
    // (1) Initialize the experiment client and implement a
    //     custom exposure tracking provider.
    class ExposureTracker: ExposureTrackingProvider {
        func track(exposure: Exposure) {
            // TODO: Implement exposure tracking
            // analytics.track(name: "$exposure", properties: [
            //     "flag_key": exposure.flagKey,
            //     "variant": exposure.variant,
            //     "experiment_key": exposure.experimentKey
            // ])
        }
    }
    let experiment = Experiment.initialize(
        apiKey: "DEPLOYMENT_KEY",
        config: ExperimentConfigBuilder()
            .exposureTrackingProvider(ExposureTracker())
            .build()
    )
    ```

#### Configuration

The SDK client can be configured once on initialization.

???config "Configuration Options"
    | <div class="big-column">Name</div> | Description | Default Value |
    | --- | --- | --- |
    | `debug` | Enable additional debug logging within the SDK. Should be set to false in production builds. | `false` |
    | `fallbackVariant` | The default variant to fall back if a variant for the provided key doesn't exist. | `{}` |
    | `initialVariants` | An initial set of variants to access. This field is valuable for bootstrapping the client SDK with values rendered by the server using server-side rendering (SSR). | `{}` |
    | `serverZone` | Select the Amplitude data center to get flags and variants from, `.US` or `.EU` | `.US` |
    | `serverUrl` | The host to fetch variants from. | `https://api.lab.amplitude.com` |
    | `flagsServerUrl` | The host to fetch local evaluation flags from. For hitting the EU data center, use `serverZone`. | `https://flag.lab.amplitude.com` |
    | `fetchTimeoutMillis` | The timeout for fetching variants in milliseconds. | `10000` |
    | `retryFetchOnFailure` | Whether to retry variant fetches in the background if the request doesn't succeed. | `true` |
    | `automaticExposureTracking` | If true, calling [`variant()`](#variant) will track an exposure event through the configured `exposureTrackingProvider`. If no exposure tracking provider is set, this configuration option does nothing.  | `true` |
    | `fetchOnStart` | If true or nil, always [fetch](#fetch) remote evaluation variants on [start](#start). If false, never fetch on start. | `true` |
    | `pollOnStart` | Poll for local evaluation flag configuration updates once per minute on [start](#start). | `true` |
    | `automaticFetchOnAmplitudeIdentityChange` | Only matters if you use the `initializeWithAmplitudeAnalytics` initialization function to seamlessly integrate with the Amplitude Analytics SDK. If `true` any change to the user ID, device ID or user properties from analytics will trigger the experiment SDK to fetch variants and update it's cache. | `false` |
    | `userProvider` | An interface used to provide the user object to `fetch()` when called. See [Experiment User](https://developers.experiment.amplitude.com/docs/experiment-user#user-providers) for more information. | `null` |
    | `exposureTrackingProvider` | Implement and configure this interface to track exposure events through the experiment SDK, either automatically or explicitly. | `null` |
    | `instanceName` | Custom instance name for experiment SDK instance. **The value of this field is case-sensitive.** | `null` |

!!!info "EU Data Center"
    If you're using Amplitude's EU data center, configure the `serverZone` option on initialization to `.EU`.

#### Integrations

If you use either Amplitude or Segment Analytics SDKs to track events into Amplitude, you'll want to set up an integration on initialization. Integrations automatically implement [provider](#providers) interfaces to enable a more streamlined developer experience by making it easier to **manage user identity** and **track exposures events**.

???amplitude "Amplitude Integration (click to open)"

    The Amplitude Experiment SDK is set up to integrate seamlessly with the Amplitude Analytics SDK.

    ```swift hl_lines="2"
    Amplitude.instance().initializeApiKey("API_KEY")
    let experiment = Experiment.initializeWithAmplitudeAnalytics(
        apiKey: "DEPLOYMENT_KEY",
        config: ExperimentConfigBuilder().build()
    )
    ```

    Note that, if you are using a custom instance name for analytics, you will need to set the same value in the `instanceName` [configuration option](#configuration) in the experiment SDK.

    Using the integration initializer will automatically configure implementations of the [user provider](#user-provider) and [exposure tracking provider](#exposure-tracking-provider) interfaces to pull user data from the Amplitude Analytics SDK and track exposure events.

    **Supported Versions**

    All generally available versions of the next-generation [Amplitude Analytics Swift](../../data/sdks/ios-swift/index.md) SDK support this integration.

    | Analytics SDK Version | Experiment SDK Version |
    | --- | --- |
    | `8.8.0+` | `1.6.0+` |

???segment "Segment Integration (click to open)"

    Experiment's integration with Segment Analytics must be configured manually. The Experiment SDK must then be configured on initialization with an instance of the the exposure tracking provider. Make sure this happens _after_ the analytics SDK has been loaded an initialized.

    ```swift title="SegmentExposureTrackingProvider"
    class SegmentExposureTrackingProvider : ExposureTrackingProvider {
        private let analytics: Analytics
        init(analytics: Analytics) {
            self.analytics = analytics
        }
        func track(exposure: Exposure) {
            analytics.track("$exposure", properties: [
                "flag_key": exposure.flagKey,
                "variant": exposure.variant,
                "experiment_key": exposure.experimentKey
            ])
        }
    }
    ```

    The Experiment SDK must then be configured on initialization with an instance of the the exposure tracking provider.

    ```swift
    let analytics = // Initialize segment analytics
    ExperimentConfig config = ExperimentConfigBuilder()
        .exposureTrackingProvider(SegmentExposureTrackingProvider(analytics))
        .build()
    let experiment = Experiment.initialize(apiKey: "<DEPLOYMENT_KEY>", config: config)
    ```

    When [fetching variants](#fetch), pass the segment anonymous ID and user ID for the device ID and user ID, respectively.

    ```swift
    let userId = SEGState.sharedInstance().userInfo.userId
    let deviceId = SEGState.sharedInstance().userInfo.anonymousId

    let user = ExperimentUserBuilder()
        .userId(userId)
        .deviceId(deviceId)
        .build()
    experiment.fetch(user: user, completion: nil)
    ```

### Start

Start the SDK by getting flag configurations from the server and fetching remote evaluation variants for the user. The SDK is ready once the completion callback is called.

```js
func start(_ user: ExperimentUser? = nil, completion: ((Error?) -> Void)? = nil)
```

| Parameter | Requirement | Description |
| --- | --- | --- |
| `user` | optional | Explicit [user](../general/data-model.md#users) information to pass with the request to fetch variants. This user information is merged with user information provided from [integrations](#integrations) via the [user provider](#user-provider), preferring properties passed explicitly to `fetch()` over provided properties. Also sets the user in the SDK for reuse. |
| `completion` | optional | The completion block, called when the SDK has finished starting. If fetch is called on start, the completion block is called after the fetch response is received. |

Call `start()` when your application is initializing, after user information is available to use to evaluate or [fetch](#fetch) variants. The provided completion block is called after loading local evaluation flag configurations and fetching remote evaluation variants.

Configure the behavior of `start()` by setting `fetchOnStart` in the SDK configuration on initialization to improve performance based on the needs of your application.

* If your application never relies on remote evaluation, set `fetchOnStart` to `false` to avoid increased startup latency caused by remote evaluation.
* If your application relies on remote evaluation, but not right at startup, you may set `fetchOnStart` to `false` and call `fetch()` separately.

=== "Amplitude Analytics"

    ```swift
    experiment.start() { error in 
        // SDK Started
    }
    ```

=== "Third Party Analytics"

    ```swift
    let user = ExperimentUserBuilder()
        .userId("user@company.com")
        .deviceId("abcdefg")
        .userProperty("premium", value: true)
        .build()
    experiment.start(user) { error in
        // SDK Started
    }
    ```

### Fetch

Fetches variants for a [user](../general/data-model.md#users) and store the results in the client for fast access. This function [remote evaluates](../general/evaluation/remote-evaluation.md) the user for flags associated with the deployment used to initialize the SDK client.

```swift
func fetch(user: ExperimentUser?, options: FetchOptions?, completion: ((ExperimentClient, Error?) -> Void)?)
```

| Parameter  | Requirement | Description |
| --- | --- | --- |
| `user` | optional | Explicit [user](../general/data-model.md#users) information to pass with the request to evaluate. This user information is merged with user information provided from [integrations](#integrations) via the [user provider](#user-provider), preferring properties passed explicitly to `fetch()` over provided properties. |
| `options` | optional | Explicit flag keys to fetch.|
| `completion` | optional | Callback when the variant fetch (success or failure). If the fetch request fails, the error is returned in the second parameter of this callback. |

Amplitude Experiment recommends calling `fetch()` during application start up so that the user gets the most up-to-date variants for the application session. Furthermore, you'll need to wait for the fetch request to return a result before rendering the user experience to avoid the interface "flickering".

```swift
let user = ExperimentUserBuilder()
    .userId("user@company.com")
    .userProperty("premium", value: true)
    .build()
experiment.fetch(user: user) { experiment, error in
    // Do something...
}
```

If you're using an [integration](#integrations) or a custom [user provider](#user-provider) then you can fetch without inputting the user.

```swift
experiment.fetch(user: nil, completion: nil)
```

???tip "Fetch When User Identity Changes"
    If you want the most up-to-date variants for the user, it's recommended that you call `fetch()` whenever the user state changes in a meaningful way. For example, if the user logs in and receives a user ID, or has a user property set which may effect flag or experiment targeting rules.

    In the case of **user properties**, Amplitude recommends passing new user properties explicitly to `fetch()` instead of relying on user enrichment prior to [remote evaluation](../general/evaluation/remote-evaluation.md). This is because user properties that are synced remotely through a separate system have no timing guarantees with respect to `fetch()`--i.e. a race.

!!!info "Timeout & Retries"
    If `fetch()` times out (default 10 seconds) or fails for any reason, the SDK client will return and retry in the background with back-off. You may configure the timeout or disable retries in the [configuration options](#configuration) when the SDK client is initialized.

---

### Variant

Access a [variant](../general/data-model.md#variants) for a [flag or experiment](../general/data-model.md#flags-and-experiments) from the SDK client's local store.

!!!info "Automatic Exposure Tracking"
    When an [integration](#integrations) is used or a custom [exposure tracking provider](#exposure-tracking-provider) is set, `variant()` will automatically track an exposure event through the tracking provider. To disable this functionality, [configure](#configuration) `automaticExposureTracking` to be `false`, and track exposures manually using [`exposure()`](#exposure).

```swift
func variant(_ key: String, fallback: Variant? = nil) -> Variant
```

| Parameter | Requirement | Description |
| --- | --- | --- |
| `key` | required | The **flag key** to identify the [flag or experiment](../general/data-model.md#flags-and-experiments) to access the variant for. |
| `fallback` | optional | The value to return if no variant was found for the given `flagKey`. |

When determining which variant a user has been bucketed into, you'll want to compare the variant `value` to a well-known string.

```swift
let variant = experiment.variant("<FLAG_KEY>")
if variant.value == "on" {
    // Flag is on
} else {
    // Flag is off
}
```

???info "Accessing the variant's payload"
    A variant may also be configured with a dynamic [payload](../general/data-model.md#variants) of arbitrary data. Access the `payload` field from the variant object after checking the variant's `value`.

    The `payload` in iOS is of type `Any?`, so cast the payload to the expected type to retrieve the value. For example, if the payload is `{"key":"value"}`:

    ```swift
    let variant = client.variant("<FLAG_KEY>")
    if variant.value == "on" {
        if let payload = variant.payload as? [String:String] {
            let value = payload["key"]
        }
    }
    ```

A `null` variant `value` means that the user hasn't been bucketed into a variant. You may use the built in **fallback** parameter to provide a variant to return if the store doesn't contain a variant for the given flag key.

```swift
let variant = experiment.variant("<FLAG_KEY>", fallback: Variant("control"))
if variant.value == "control" {
    // Control
} else if variant.value == "treatment" {
    // Treatment
}
```

---

### All

Access all [variants](../general/data-model.md#variants) stored by the SDK client.

```swift
func all() -> [String:Variant]
```

---

### Clear

Clear all [variants](../general/data-model.md#variants) in the cache and storage.

```swift
func clear()
```

You can call `clear` after user logout to clear the variants in cache and storage.

```swift
experiment.clear()
```

---

### Exposure

Manually track an [exposure event](../general/experiment-event-tracking.md#exposure-events) for the current variant of the given flag key through configured [integration](#integrations) or custom [exposure tracking provider](#exposure-tracking-provider). Generally used in conjunction with setting the `automaticExposureTracking` [configuration](#configuration) optional to `false`.

```swift
func exposure(key: String)
```

| Parameter | Requirement | Description |
| --- | --- | --- |
| `key` | required | The **flag key** to identify the [flag or experiment](../general/data-model.md#flags-and-experiments) variant to track an [exposure event](../general/experiment-event-tracking.md#exposure-events) for. |

```swift
let variant = experiment.variant("<FLAG_KEY>")

// Do other things...

experiment.exposure("<FLAG_KEY>")
if variant.value == "control" {
    // Control
} else if variant.value == "treatment" {
    // Treatment
}
```

---

## Providers

!!!tip "Integrations"
    If you use Amplitude or Segment analytics SDKs along side the Experiment Client SDK, Amplitude recommends you use an [integration](#integrations) instead of implementing custom providers.

Provider implementations enable a more streamlined developer experience by making it easier to manage user identity and track exposures events.

### User provider

The user provider is used by the SDK client to access the most up-to-date user information only when it's needed (for example: when [`fetch()`](#fetch) is called). This provider is optional, but helps if you have a user information store already set up in your application. This way, you don't need to manage two separate user info stores in parallel, which may result in a divergent user state if the application user store is updated and experiment isn't (or via versa).
<!-- vale off-->
```swift title="ExperimentUserProvider"
protocol ExperimentUserProvider {
    func getUser() -> ExperimentUser
}
```
<!-- vale on-->
To use your custom user provider, set the `userProvider` [configuration](#configuration) option with an instance of your custom implementation on SDK initialization.

```swift
let config = ExperimentConfigBuilder()
    .userProvider(CustomUserProvider())
    .build()
let experiment = Experiment.initialize(apiKey: "<DEPLOYMENT_KEY>", config: config)
```

### Exposure tracking provider

Implementing an exposure tracking provider is highly recommended. [Exposure tracking](../general/experiment-event-tracking.md#exposure-events) increases the accuracy and reliability of experiment results and improves visibility into which flags and experiments a user is exposed to.

```swift title="ExposureTrackingProvider"
protocol ExposureTrackingProvider {
    func track(exposure: Exposure)
}
```

The implementation of `track()` should track an event of type `$exposure` (a.k.a name) with two event properties, `flag_key` and `variant`, corresponding to the two fields on the `Exposure` object argument. Finally, the event tracked must eventually end up in Amplitude Analytics for the same project that the [deployment] used to [initialize](#initialize) the SDK client lives within, and for the same user that variants were [fetched](#fetch) for.

To use your custom user provider, set the `exposureTrackingProvider` [configuration](#configuration) option with an instance of your custom implementation on SDK initialization.

```swift
ExperimentConfig config = ExperimentConfigBuilder()
    .exposureTrackingProvider(CustomExposureTrackingProvider(analytics))
    .build()
let experiment = Experiment.initialize(apiKey: "<DEPLOYMENT_KEY>", config: config)
```
