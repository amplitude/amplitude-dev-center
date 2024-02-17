---
title: Experiment PHP SDK (Beta)
description: Official documentation for Amplitude Experiment's server-side PHP SDK implementation.
icon: simple/php
---

Official documentation for Amplitude Experiment's server-side PHP SDK implementation.

![Packagist](https://img.shields.io/packagist/v/amplitude/experiment-php-server.svg)

!!!beta "SDK Resources"
     [:material-github: GitHub](https://github.com/amplitude/experiment-php-server) · [:material-code-tags-check: Releases](https://github.com/amplitude/experiment-php-server/releases)

### Install

!!!info "PHP version compatibility"

    The PHP Server SDK works with PHP 7.4+.

Install the PHP Server SDK with composer.

=== "php"

    ```bash
    composer require amplitude/experiment-php-server
    ```

## Remote evaluation

This SDK supports and uses [remote evaluation](../general/evaluation/remote-evaluation.md) to fetch variants for users.

!!!tip "Quick Start"

    1. [Initialize the experiment client](#initialize-remote-evaluation)
    2. [Fetch variants for the user](#fetch)
    3. [Access a flag's variant](#fetch)

    ```php
    <?php
    // (1) Initialize the experiment client
    $experiment = new \AmplitudeExperiment\Experiment();
    $client = $experiment->initializeRemote('<DEPLOYMENT_KEY>');

    // (2) Fetch variants for a user
    $user = \AmplitudeExperiment\User::builder()
        ->deviceId('abcdefg')
        ->userId('user@company.com')
        ->userProperties(['premium' => true]) 
        ->build();
    $variants = $client->fetch($user);

    // (3) Access a flag's variant
    $variant = $variants['FLAG_KEY'] ?? null;
    if ($variant) {
        if ($variant->value == 'on') {
            // Flag is on
        } else {
            // Flag is off
        }
    }
    ```

    **Not getting the expected variant result for your flag?** Make sure your flag [is activated](../guides/getting-started/create-a-flag.md#activate-the-flag), has a [deployment set](../guides/getting-started/create-a-flag.md#add-a-deployment), and has [users allocated](../guides/getting-started/create-a-flag.md#configure-targeting-rules).

### Initialize remote evaluation

Configure the SDK to initialize on server startup. The [deployment key](../general/data-model.md#deployments) argument you pass into the `apiKey` parameter must live within the same project that you send analytics events to.

```php
<?php
initializeRemote(string $apiKey, ?RemoteEvaluationConfig $config = null): RemoteEvaluationClient
```

| Parameter | Requirement | Description |
| --- | --- | --- |
| `apiKey` | required | The [deployment key](../general/data-model.md#deployments) which authorizes fetch requests and determines which flags to evaluate for the user. |
| `config` | optional | The client [configuration](#configuration) used to customize SDK client behavior. |

#### Configuration

You can configure the SDK client on initialization.

???config "Configuration Options"
    | <div class="big-column">Name</div>  | Description | Default Value |
    | --- | --- | --- |
    | `serverUrl` | The host to fetch variants from. | `https://api.lab.amplitude.com` |
    | `logger` | Set to use custom logger. If not set, a [default logger](#custom-logger) is used. | `null` |
    | `logLevel` | The [log level](#log-level) to use for the logger. | `LogLevel::ERROR` |
    | `httpClient` | The underlying [HTTP client](#custom-http-client) to use for requests, if this is not set, a [default](#guzzlehttpclient) HTTP client will be used. | `null` |
    | `guzzleClientConfig` | The configuration for the underlying default `GuzzleHttpClient` (if used). | [defaults](#guzzlehttpclient) |

!!!info "EU Data Center"
    If you use Amplitude's EU data center, set the `serverUrl` option on initialization to `https://api.lab.eu.amplitude.com`

### Fetch

Fetches variants for a [user](../general/data-model.md#users) and returns the results. This function [remote evaluates](../general/evaluation/remote-evaluation.md) the user for flags associated with the deployment used to initialize the SDK client.

```php
<?php
fetch(User $user, array $flagKeys = []): array<Variant>
// An array of variants is returned on success, an empty array is returned on failure
```

| Parameter  | Requirement | Description |
| --- | --- | --- |
| `user` | required | The [user](../general/data-model.md#users) to remote fetch variants for. |
| `flagKeys` | optional | Specific flags or experiments to evaluate. If empty, Amplitude evaluates all flags and experiments. |

```php
<?php
$user = \AmplitudeExperiment\User::builder()
    ->deviceId('abcdefg')
    ->userId('user@company.com')
    ->userProperties(['premium' => true])
    ->build();
$variants = $client->fetch($user);
```

After fetching variants for a user, you may to access the variant for a specific flag.

```php
<?php
$variant = $variants['FLAG-KEY'] ?? null;
if ($variant) {
    if ($variant->value == 'on') {
        // Flag is on
    } else {
        // Flag is off
    }
}
```

## Local evaluation

Implements evaluation of variants for a user through [local evaluation](../general/evaluation/local-evaluation.md). If you plan to use local evaluation, you should [understand the tradeoffs](../general/evaluation/local-evaluation.md#targeting-capabilities).

!!!note "Local Evaluation Mode"
    The local evaluation client can only evaluate flags which are [set to local evaluation mode](../guides/create-local-evaluation-flag.md).

!!!tip "Quick Start"

    1. [Initialize the local evaluation client.](#initialize-local-evaluation)
    2. [Fetch flag configs for the local evaluation client.](#refreshFlagConfigs)
    3. [Evaluate a user.](#evaluate)

    ```php
    <?php
    // (1) Initialize the experiment client
    $experiment = new \AmplitudeExperiment\Experiment();
    $client = $experiment->initializeLocal('<DEPLOYMENT_KEY>');

    // (2) Fetch flags for the local evaluation client.
    $client->refreshFlagConfigs();

    // (3) Evaluate a user.
    $user = \AmplitudeExperiment\User::builder()
        ->deviceId('abcdefg')
        ->userId('user@company.com')
        ->userProperties(['premium' => true]) 
        ->build();

    $variants = $client->evaluate($user);
    ```

    **Not getting the expected variant result for your flag?** Make sure your flag [is activated](../guides/getting-started/create-a-flag.md#activate-the-flag), has a [deployment set](../guides/getting-started/create-a-flag.md#add-a-deployment), and has [users allocated](../guides/getting-started/create-a-flag.md#configure-targeting-rules).

### Initialize local evaluation

For more information, see [Local Evaluation](../general/evaluation/local-evaluation.md).

!!!warning "Server Deployment Key"
    [Initialize](#initialize-local-evaluation) the local evaluation client with a server [deployment](../general/data-model.md#deployments) key to access local evaluation flag configurations.

```php
initializeLocal(string $apiKey, ?LocalEvaluationConfig $config = null): LocalEvaluationClient
```

| Parameter | Requirement | Description |
| --- | --- | --- |
| `apiKey` | required | The server [deployment key](../general/data-model.md#deployments) which authorizes fetch requests and determines which flags to evaluate for the user. |
| `config` | optional | The client [configuration](#configuration_1) used to customize SDK client behavior. |

#### Configuration

You can configure the SDK client on initialization.

???config "Configuration Options"
    | <div class="big-column">Name</div> | Description | Default Value |
    | --- | --- | --- |
    | `serverUrl` | The host to fetch flag configurations from. | `https://api.lab.amplitude.com` |
    | `logger` | Set to use custom logger. If not set, a [default logger](#custom-logger) is used. | `null` |
    | `logLevel` | The [log level](#log-level) to use for the logger. | `LogLevel::ERROR` |
    | `httpClient` | The underlying [HTTP client](#custom-http-client) to use for requests, if this is not set, a [default](#guzzlehttpclient) HTTP client will be used. | `null` |
    | `guzzleClientConfig` | The configuration for the underlying default `GuzzleHttpClient` (if used). | [defaults](#guzzlehttpclient) |
    | `bootstrap` | Bootstrap the client with an array of flag key to flag configuration | `[]` |
    | [`assignmentConfig`](#assignment-tracking) | Configuration for automatically tracking assignment events after an evaluation. | `null` |

!!!info "EU Data Center"
    If you use Amplitude's EU data center, configure the `serverUrl` option on initialization to `https://api.lab.eu.amplitude.com`

### refreshFlagConfigs

Fetch up-to-date local evaluation mode flag configs for [evaluation](#evaluate).

```php
refreshFlagConfigs(): void
```

Call `refreshFlagConfigs()` to ensure that flag configs are up-to-date before you use [`evaluate()`](#evaluate)

```php
<?php
$client->refreshFlagConfigs();
```

### getFlagConfigs

Return flag configs currently used in the client.

```php
getFlagConfigs(): array
```

Flag configs returned can be used to reduce start up time by [bootstrapping](#configuration_1) the local evaluation client.

```php
<?php
$client->getFlagConfigs();
```

### Evaluate

Executes the [evaluation logic](../general/evaluation/implementation.md) using the flags fetched on [`refreshFlagConfigs()`](#refreshFlagConfigs). Give `evaluate()` a user object argument. Optionally pass an array of flag keys if you require only a specific subset of required flag variants.

!!!tip "Automatic Assignment Tracking"
    Set [`assignmentConfig`](#configuration_1) to automatically track an assignment event to Amplitude when you call `evaluate()`.

```php
evaluate(User $user, array $flagKeys = []): array
```

| Parameter | Requirement | Description |
| --- | --- | --- |
| `user` | required | The [user](../general/data-model.md#users) to evaluate. |
| `flagKeys` | optional | Specific flags or experiments to evaluate. If empty, Amplitude evaluates all flags and experiments. |

```php
<?php
// The user to evaluate
$user = \AmplitudeExperiment\User::builder()
        ->deviceId('abcdefg')
        ->build();

// Evaluate all flag variants
$allVariants = $client->evaluate($user);

// Evaluate a specific subset of flag variants
$specificVariants = $client->evaluate($user, [
  'my-local-flag-1',
  'my-local-flag-2',
]);

// Access a flag's variant
$variant = $allVariants['FLAG_KEY'] ?? null;
if ($variant) {
    if ($variant->value == 'on') {
        // Flag is on
    } else {
        // Flag is off
    }
}
```

### Assignment tracking

You can configure the local evaluation client to send [assignment events](../../general/experiment-event-tracking/#assignment-events) to Amplitude.

???config "Configuration Options"
    | <div class="big-column">Name</div> | Description | Default Value |
    | --- | --- | --- |
    | `assignmentTrackingProvider` | The AssignmentTrackingProvider used to send assignment events. | *required* |
    | `cacheCapacity` | The maximum number of assignments stored in the assignment cache. | `65536` |
    | `apiKey` | The analytics API key. Not to be confused with the experiment deployment key. | *required* |
    | `minIdLength` | The minimum length of `userId` and `deviceId`. | `5` |

#### AssignmentTrackingProvider

The local evaluation client uses an assignment tracking provider to send assignment events to Amplitude. Amplitude provides a default assignment tracking provider, but this is best used for testing due to its synchronous nature. Amplitude recommends that you use a custom provider for increased flexibility and performance.

```php title="AssignmentTrackingProvider"
<?php
interface AssignmentTrackingProvider {
  public function track(Assignment $assignment): void;
}
```

The local evaluation client calls `track()` when it determines there are untracked assignment events. It compares the resulting assignment from `evaluate()` with the assignment cache and tracks it if it's not in the cache.

| Parameter | Requirement | Description |
| --- | --- | --- |
| `assignment` | required | The object representing an Experiment assignment event |

#### DefaultAssignmentTrackingProvider

The default assignment tracking provider is a basic implementation of the interface which uses the internal `Amplitude` package to send assignment events through synchronous HTTP requests.

```php title="DefaultAssignmentTrackingProvider"
<?php
class DefaultAssignmentTrackingProvider implements AssignmentTrackingProvider {
  public function __construct(Amplitude $amplitude);
}
```

???config "Configuration Options"

    **Amplitude**

    | <div class="big-column">Name</div> | Description | Default Value |
    | --- | --- | --- |
    | `apiKey` | The analytics API key. Not to be confused with the experiment deployment key. | *required* |
    | `config` | Configuration options | `null` |

    **AmplitudeConfig**

    | <div class="big-column">Name</div> | Description | Default Value |
    | --- | --- | --- |
    | `flushQueueSize` | Events wait in the buffer and are sent in a batch. Experiment flushes the buffer when the number of events reaches the `flushQueueSize`. | `200` |
    | `minIdLength` | The minimum length of `userId` and `deviceId`. | `5` |
    | `serverZone` | The server zone of the projects. Supports `EU` and `US`. For EU data residency, Change to `EU`. | `US` |
    | `serverUrl` | The API endpoint URL that events are sent to. Automatically selected by `serverZone` and `useBatch`. If this field is set with a string value instead of `null`, then `serverZone` and `useBatch` are ignored and the string value is used. | `https://api2.amplitude.com/2/httpapi` |
    | `useBatch` | Whether to use [batch API](../../../analytics/apis/batch-event-upload-api/#batch-event-upload). By default, the SDK will use the default `serverUrl`. | `false` |
    | `httpClient` | The underlying [HTTP client](#custom-http-client) to use for requests, if this is not set, a [default](#guzzlehttpclient) HTTP client will be used. | `null` |
    | `guzzleClientConfig` | The configuration for the underlying default `GuzzleHttpClient` (if used). | [defaults](#guzzlehttpclient) |
    | `logger` | Set to use custom logger. If not set, a [default logger](#custom-logger) is used. | `null` |
    | `logLevel` | The [log level](#log-level) to use for the logger. | `LogLevel::ERROR` |

```php
<?php
$config = \AmplitudeExperiment\Amplitude\AmplitudeConfig::builder()
          ->useBatch(true)
          ->minIdLength(10)
          ->build();
$amplitude = new \AmplitudeExperiment\Amplitude\Amplitude('<API_Key>', $config);
$defaultAssignmentTrackingProvider = new \AmplitudeExperiment\Assignment\DefaultAssignmentTrackingProvider($amplitude);
```

## Custom Logger

Local and remote evaluation clients can be configured to use a custom logger which implements the PSR logger interface, otherwise a default `error_log`-based logger is used.

### Log level

The following log levels are used by the SDK:

| Level | Description |
| --- | --- |
| `NO_LOG` | Turn off logging |
| `ERROR` | Error-level messages are logged |
| `DEBUG` | Debug and error-level messages are logged |

## Custom HTTP Client

Local and remote evaluation clients can be configured to use a custom HTTP client which implements the [HttpClientInterface](#httpclientinterface), otherwise a default [Guzzle-based HTTP client](#guzzlehttpclient) is used.

### HttpClientInterface

| Method | Return Type | Description |
| --- | --- | --- |
| `getClient` | `Psr\Http\Client\ClientInterface` | Return the underlying PSR HTTP Client. |
| `createRequest` | `Psr\Http\Message\RequestInterface` | Return a PSR Request to be sent by the underlying PSR HTTP Client. |

### GuzzleHttpClient

The default Guzzle client can be configured via the `guzzleClientConfig` option in [`RemoteEvaluationConfig`](#configuration) and [`LocalEvaluationConfig`](#configuration_1).

???config "AssignmentConfig Configuration Options"
    | <div class="big-column">Name</div>  | Description | Default Value |
        | --- | --- | --- |
    | `timeoutMillis` | The timeout for requests in milliseconds. This timeout applies to the initial request, not subsequent retries. | `10000` |
    | `retries` | The number of retries to attempt if a request fails. | `8` |
    | `retryBackoffMinMillis` | The minimum (initial) backoff after a request fails. This delay scales according to the `retryBackoffScalar` setting. | `500` |
    | `retryBackoffMaxMillis` | The maximum backoff between retries. If the scaled backoff becomes greater than the maximum, Experiment uses the  maximum for all subsequent requests. | `10000` |
    | `retryBackoffScalar` | Scales the minimum backoff exponentially. | `1.5` |
    | `retryTimeoutMillis` | The request timeout for retrying requests. | `10000` |

```php
<?php
// Configure the default Guzzle client
$config = \AmplitudeExperiment\RemoteEvaluationConfig::builder()
          ->guzzleClientConfig(['timeoutMillis' => 5000, 'retries' => 10])
          ->build();
```

## Access Amplitude cookies

If you use the Amplitude Analytics SDK on the client-side, the PHP server SDK provides an `AmplitudeCookie` class with convenience functions for parsing and interacting with the Amplitude identity cookie. This helps ensure that the Device ID on the server matches the Device ID set on the client, especially if the client hasn't yet generated a Device ID.

```php
<?php
// Grab amp device id if present
$ampCookieName = AmplitudeCookie::cookieName('amplitude-api-key');
$deviceId = null;

if (!empty($_COOKIE[$ampCookieName])) {
    $parsedCookie = AmplitudeCookie::parse($_COOKIE[$ampCookieName]);
    $deviceId = $parsedCookie['deviceId'];
}

if ($deviceId === null) {
    // deviceId doesn't exist, set the Amplitude Cookie
    $deviceId = bin2hex(random_bytes(16));
    $ampCookieValue = AmplitudeCookie::generate($deviceId);
    setcookie($ampCookieName, $ampCookieValue, [
        'domain' => '.your-domain.com', // this should be the same domain used by the Amplitude JS SDK
        'httponly' => false,
        'secure' => false,
    ]);
}
```