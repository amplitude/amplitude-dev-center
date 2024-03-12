---
title: Candu Event Streaming
description: Stream Amplitude events to Candu
---

Candu helps you build, iterate, and personalize your in-app content experiences. This integration enables you to stream events and users updates from to Amplitude to Candu.

!!!info "This integration is managed by Candu"

  Contact the [Candu support team](https://docs.candu.ai/en/) with any questions about this integration.

## Considerations

- You must enable this integration in each Amplitude project you want to use it in.
- You need a Candu account to enable this integration.
- Amplitude sends selected user and event properties along with the event.

## Setup

### Prerequisites

To configure an Event Streaming integration from Amplitude to Candu, you must fulfill the following prerequisites from Candu:

- **A Candu Account:** You must have a Candu account to use this integration. Contact Candu to learn more.
- **Candu API Key:** Candu requires an API key to send data to Candu.

### Candu setup

1. Log in to your My Candu account.
2. Navigate to the **Credentials** tab to find and copy your account's API Key.

### Amplitude setup

1. In Amplitude, navigate to **Data Destinations**, then find **Candu - Event Stream**.
2. Enter a sync name, then click **Create Sync**.
3. Toggle Status from **Disabled** to **Enabled**.
4. Paste your **API Key (Access Token from the Candu platform)**.
5. (Optional) In the **Create & Update users** section, enable the toggle if you want to send users and their properties in real-time whenever Amplitude creates a user or updates the user property.
6. In the **Send Events** section, enable the **Events are sent to Candu** toggle to stream events to Candu. When enabled, Amplitude forwards events to Candu when they're ingested. Events aren't sent on a schedule or on demand using this integration.
7. In the **Select and filter events** section choose which events you want to send. Choose only the events you need in Candu. This integration doesn't support[Transformed events](https://www.google.com/url?q=https://help.amplitude.com/hc/en-us/articles/5913315221915-Transformations-Retroactively-modify-your-event-data-structure%23:~:text%3DAmplitude%2520Data%27s%2520transformations%2520feature%2520allows,them%2520to%2520all%2520historical%2520data.&sa=D&source=docs&ust=1692341974637179&usg=AOvVaw1BdAYfjzWTy1y9u94STUaQ).
8. Enable the destination and **Save** to finish.
