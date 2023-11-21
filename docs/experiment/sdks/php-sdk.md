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
    $variants = $client->fetch($user)>wait();

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

!!!info "Timeout & Retry Configuration"
    Configure the timeout and retry options to best fit your performance requirements.
    ```php
    <?php
    $experiment = new \AmplitudeExperiment\Experiment();
    $config = \AmplitudeExperiment\Remote\RemoteEvaluationConfig::builder()
                ->fetchTimeoutMillis(500)
                ->fetchRetries(1)
                ->fetchRetryBackoffMinMillis(0)
                ->fetchRetryTimeoutMillis(500)
                ->build();
    $client = $experiment->initializeRemote('<DEPLOYMENT_KEY>', $config);
    ```

#### Configuration

You can configure the SDK client on initialization.

???config "Configuration Options"
    | <div class="big-column">Name</div>  | Description | Default Value |
    | --- | --- | --- |
    | `debug` | Enable additional debug logging. | `false` |
    | `serverUrl` | The host to fetch variants from. | `https://api.lab.amplitude.com` |
    | `fetchTimeoutMillis` | The timeout for fetching variants in milliseconds. This timeout applies to the initial request, not subsequent retries. | `10000` |
    | `fetchRetries` | The number of retries to attempt if a request to fetch variants fails. | `8` |
    | `fetchRetryBackoffMinMillis` | The minimum (initial) backoff after a request to fetch variants fails. This delay scales according to the `fetchRetryBackoffScalar` setting. | `500` |
    | `fetchRetryBackoffMaxMillis` | The maximum backoff between retries. If the scaled backoff becomes greater than the maximum, Experiment uses the  maximum for all subsequent requests. | `10000` |
    | `fetchRetryBackoffScalar` | Scales the minimum backoff exponentially. | `1.5` |
    | `fetchRetryTimeoutMillis` | The request timeout for retrying variant fetches. | `10000` |

!!!info "EU Data Center"
    If you use Amplitude's EU data center, set the `serverUrl` option on initialization to `https://api.lab.eu.amplitude.com`

### Fetch

Fetches variants for a [user](../general/data-model.md#users) and returns the results. This function [remote evaluates](../general/evaluation/remote-evaluation.md) the user for flags associated with the deployment used to initialize the SDK client.

```php
<?php
fetch(User $user): PromiseInterface
// Upon resolution of the promise, an array of variants is returned on success, an empty array is returned on failure
```

| Parameter  | Requirement | Description |
| --- | --- | --- |
| `user` | required | The [user](../general/data-model.md#users) to remote fetch variants for. |

```php
<?php
$user = \AmplitudeExperiment\User::builder()
    ->deviceId('abcdefg')
    ->userId('user@company.com')
    ->userProperties(['premium' => true])
    ->build();
$variants = $client.fetch($user)->wait();
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
    2. [Start the local evaluation client.](#start)
    3. [Evaluate a user.](#evaluate)

    ```php
    <?php
    // (1) Initialize the experiment client
    $experiment = new \AmplitudeExperiment\Experiment();
    $client = $experiment->initializeLocal('<DEPLOYMENT_KEY>');

    // (2) Start the local evaluation client.
    $client->start()->wait();

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
    | `debug` | Set to `true` to enable debug logging. | `false` |
    | `serverUrl` | The host to fetch flag configurations from. | `https://api.lab.amplitude.com` |
    | `bootstrap` | Bootstrap the client with an array of flag key to flag configuration | `[]` |

!!!info "EU Data Center"
    If you use Amplitude's EU data center, configure the `serverUrl` option on initialization to `https://api.lab.eu.amplitude.com`

### Start

Fetch local evaluation mode flag configs for [evaluation](#evaluate).

```php
start(): PromiseInterface
```

Await the result of `start()` to ensure that flag configs are ready for use before you call [`evaluate()`](#evaluate)

```php
<?php
$client->start()->wait();
```

### Evaluate

Executes the [evaluation logic](../general/evaluation/implementation.md) using the flags fetched on [`start()`](#start). Give `evaluate()` a user object argument. Optionally pass an array of flag keys if you require only a specific subset of required flag variants.

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