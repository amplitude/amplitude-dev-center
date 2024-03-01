---
title: Evaluation Proxy
description: Containerized service to enhance local evaluation running in your infrastructure.
icon: simple/docker
---

!!!warning "The evaluation proxy is under active development. APIs are unstable and may change before general availability."

!!!beta "Beta Resources"
    [:material-github: GitHub](https://github.com/amplitude/evaluation-proxy) · [:material-code-tags-check: Releases](https://github.com/amplitude/evaluation-proxy/releases) · [:simple-docker: Docker Image](https://hub.docker.com/r/amplitudeinc/evaluation-proxy) · [:simple-helm: Helm Chart](https://github.com/amplitude/evaluation-proxy-helm)
<br />
The Evaluation Proxy is a Service to enable, enhance, and optimize [local evaluation](../general/evaluation/local-evaluation.md) running within your infrastructure.

![Architectural diagram showing how the evaluation proxy interacts with your services and Amplitude.](../../../assets/images/experiment/evaluation-proxy.drawio.svg)

:material-check-circle: **Enable local evaluation on unsupported platforms**:<br /> Use remote [Evaluation APIs](../apis/evaluation-api.md) and [SDKs](../index.md#sdks) to run local evaluation in your infrastructure.

:material-check-circle: **Automatically track assignment events for local evaluations**:<br /> Identical assignment events are deduplicated for 24 hours.

:material-check-circle: **Enhance local evaluation with large cohort targeting**:<br /> Targeted cohorts are synced hourly to the Evaluation Proxy and added to the user prior to evaluation.

## Configuration

The evaluation proxy is either configured via a `yaml` file (recommended, more configuration options), or using environment variables.

The default location for the configuration yaml file is `/etc/evaluation-proxy-config.yaml`. You may also configure the file location using the `PROXY_CONFIG_FILE_PATH` environment variable.

* [`projects`](#projects) (required)
* [`configuration`](#configuration-1) (optional).

!!!tip "Recommended configuration"

    Replace the fields in the configuration with values specific to your account/infrastructure.

    ```yaml
    projects:
      - apiKey: "YOUR API KEY"
        secretKey: "YOUR SECRET KEY"
        managementKey: "YOUR MANAGEMENT API KEY"

    configuration:
      redis:
        uri: "YOUR REDIS URI" # e.g. "redis://localhost:6379"
    ```

???config "Environment variable configuration (click to open)"

    Environment configuration can only configure a single project. Environment variable configuration is only considered if the configuration file is not found.

    | Environment Variable | Description |
    | --- | --- |
    | `AMPLITUDE_API_KEY` | The project's [API key](../../guides/amplitude-keys-guide.md#api-key). |
    | `AMPLITUDE_SECRET_KEY` | The project's [secret key](../../guides/amplitude-keys-guide.md#secret-key). |
    | `AMPLITUDE_EXPERIMENT_MANAGEMENT_API_KEY` | <span style="max-width:450px;display:inline-block">The [Experiment management API key](../../guides/amplitude-keys-guide.md#management-api-key). Must be created for the same project as the configured API and secret key. Used to automatically access and update deployments used for the project.</span> |
    | `AMPLITUDE_REDIS_URI` | Optional. The entire URI to connect to Redis. Include the protocol, host, port, and optional username, password, and path (for example `redis://localhost:6379`). |
    | `AMPLITUDE_REDIS_PREFIX` | Optional. The prefix to connect  |
    | `AMPLITUDE_SERVER_URL` | Optional. The server URL, including protocol and host, to fetch flags from. |
    | `AMPLITUDE_COHORT_SERVER_URL` | Optional. The server URL, including protocol and host, to download cohorts from. |

| Field | Type | Description |
| --- | --- | --- |
| `projects` | array | Required. See [`projects`](#projects). |
| `configuration` | object | Optional. See [`configuration`](#configuration-1) |

### `projects`

A required array of objects with the following fields, all which are required.

| <div class="big-column">Field</div> | <div style="max-width:450px;display:inline-block">Description</div> |
| --- | --- |
| `id` | The project's ID. Found in the project settings. |
| `apiKey` | The project's [API key](../../guides/amplitude-keys-guide.md#api-key). |
| `secretKey` | The project's [secret key](../../guides/amplitude-keys-guide.md#secret-key). |
| `managementKey` | The [Experiment management API key](../../guides/amplitude-keys-guide.md#management-api-key). Must be created for the same project as the configured API and secret key. Used to automatically access and update deployments used for the project. |

### `configuration`

An optional object of extra configuration.

| <div class="big-column">Field</div> | <div style="max-width:450px;display:inline-block">Description</div> |
| --- | --- |
| `redis` | Optional (Recommended). See [`redis`](#redis). Configure the proxy to use redis as persistent storage. |
| `flagSyncIntervalMillis` | Optional. The polling interval to update flag configurations (default `10000`). |
| `maxCohortSize` | Optional. The maximum size of targeted cohorts that the proxy can download (default `2147483647`). |
| `serverUrl` | Optional. The server URL, including protocol and host, to fetch flags from. (default `https://api.lab.amplitude.com`) |
| `cohortServerUrl` | Optional. The server URL, including protocol and host, to download cohorts from. (default `https://cohort.lab.amplitude.com`) |

!!!info "EU Data Residency"
    To use the evaluation proxy with the EU data center, set the [`serverUrl`](#configuration-1) and [`cohortServerUrl`](#configuration-1) configurations to hit the EU data center endpoints:
    ```yaml
    configuration:
      # Other configurations...
      serverUrl: "https://api.lab.eu.amplitude.com"
      cohortServerUrl: "https://cohort.lab.eu.amplitude.com"
    ```

#### `redis`

Configure the evaluation proxy to use Redis as a persistent storage. Highly recommended to enable the evaluation proxy to run efficiently.

| <div class="big-column">Field</div> | <div style="max-width:450px;display:inline-block">Description</div> |
| --- | --- |
| `uri` | Required. The full URI to connect to Redis with. Include the protocol, host, port, and optional username, password, and path. |
| `readOnlyUri` | Optional. Optional URI to connect to read only replicas for high scaling high volume reads to Redis read replicas. |
| `prefix` | Optional. A prefix for all keys saved by the evaluation proxy (default `amplitude`). |

## Deployment

The evaluation proxy is stateless, and should be deployed with multiple instances behind a load balancer for high availability and scalability.

For example, a kubernetes deployment with greater than one replica.

### Kubernetes

Use the evaluation proxy [Helm chart](https://github.com/amplitude/evaluation-proxy-helm) to install the proxy service on kubernetes or generate the files needed to deploy the service manually. The repository also contains an [example of running the evaluation proxy on kubernetes](https://github.com/amplitude/evaluation-proxy-helm/tree/main/example) locally using `minikube`.

#### Helm

##### Add helm repo

```bash
helm repo add \
    evaluation-proxy-helm https://amplitude.github.io/evaluation-proxy-helm
```

##### Configure `values.yaml`

Configure the chart values. The recommended approach to configuring and installing the helm chart is using a values.yaml configuration file.

The chart's `evaluationProxy` value contents exactly match the evaluation proxy's configuration file fields.

```yaml title="values.yaml"
evaluationProxy:
  # At least one project is required.
  projects:
    - apiKey: "YOUR API KEY"
      secretKey: "YOUR SECRET KEY"
      managementKey: "YOUR MANAGEMENT API KEY"
  configuration: {}
#    redis:
#      uri: "redis://redis-master.default.svc.cluster.local:6379"
```

##### Install helm chart

```bash
helm install -f values.yaml \
    evaluation-proxy evaluation-proxy-helm/evaluation-proxy
```

### Docker

[![docker image version](https://img.shields.io/docker/v/amplitudeinc/evaluation-proxy?color=blue&label=docker&logo=docker&logoColor=white)](https://hub.docker.com/r/amplitudeinc/evaluation-proxy)

You may run [the docker image](https://hub.docker.com/r/amplitudeinc/evaluation-proxy) directly. First, create a [configuration](#configuration) file, then run the docker image mounting the file as a volume to the expected directory in the container.

```bash
docker run \
    -v CONFIG_FILE_PATH:/etc/evaluation-proxy-config.yaml \
    amplitudeinc/evaluation-proxy
```

!!!tip "Docker compose example"
    The [evaluation-proxy GitHub repository](https://github.com/amplitude/evaluation-proxy) also contains an example using `docker compose` to run the proxy alongside a local Redis image.

## Evaluation

The Evaluation Proxy exposes remote [Evaluation API](../apis/evaluation-api.md) and [SDK](../index.md#sdks) endpoints to run local evaluation within your cluster. This is useful to enable platforms and languages which aren't supported by local evaluation SDKs. As an added benefit, fetch requests made to the evaluation proxy can target cohorts of users, and have assignment events tracked automatically to Amplitude.

**You must send requests to the service using `http` on port `3546`.**

!!!example "Kubernetes"
    A Kubernetes deployed Evaluation Proxy service (named `evaluation-proxy`) running within a kubernetes namespace `main` is from within the cluster at: `http://evaluation-proxy.main.svc.cluster.local:3546`
