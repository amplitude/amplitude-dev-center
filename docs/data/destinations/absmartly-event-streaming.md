---
title: ABsmartly Event Streaming
description: Stream Amplitude events to ABsmartly
---

!!!beta "This feature is in open beta"

    This feature is in open beta and is in active development. Contact the [ABsmartly Support team](https://www.ABsmartly.com) for support with this integration.

[ABsmartly](https://absmartly.com/) ABsmartly is a data-driven experimentation platform for A/B testing and data analysis, offering unlimited tests, accelerated results, seamless integration, real-time reporting, and robust analytical tools. It enables businesses to optimize digital performance and make data-driven decisions with ease.

## Considerations

- You must enable this integration in each Amplitude project you want to use it in.
- You need an ABsmartly account to enable this integration.
- Amplitude sends selected event properties along with the event.

## Setup

### ABsmartly setup

1. Log in to your ABsmartly account.
2. Navigate to the Settings page to obtain your **Access Token**.

### Amplitude setup

1. In Amplitude, navigate to **Data Destinations**, then find **ABsmartly - Event Stream**.
2. Enter a sync name, then click **Create Sync**.
3. Toggle Status from **Disabled** to **Enabled**.
4. Paste your **Collector Endpoint**.
5. Paste your **SDK API Key (Access Token from the ABsmartly platform)**.
6. Toggle the **Send events filter** to select the events to send. ABsmartly recommends choosing the events that are most important to your use case. 
7. When finished, enable the destination and Save.

### Use cases

1. **Real-time Event Tracking:** With event data from Amplitude sent to ABsmartly, you can track goals in real-time based on specific user interactions. For example, when a user performs an action like adding items to their cart or reaching a certain engagement threshold, ABsmartly can receive those events in real-time. This approach enables you to see data in your ABsmartly dashboards with just a few seconds of lag.
2. **Experiment Validation with Event Data:** By sending experiment data and variant assignments from ABsmartly to Amplitude, you can validate experiment results by creating a custom experiment report. This integration allows you to cross-reference user events in Amplitude with the data collected during experiments. You can then analyze how users' actions change based on the variants they were exposed to, allowing you to create custom experiment reports. This validation process helps ensure that your experiments are truly effective in driving desired outcomes.
3. **Conversion Rate Optimization:** To optimize conversion rates for specific user actions like sign-ups or purchases, leverage event data from Amplitude to inform your experiments. ABsmartly can use this data to run experiments that target users who have shown a particular behavior or completed a specific event. By tailoring experiments based on these user actions, you can work towards improving conversion rates and enhancing the overall user experience, ultimately leading to increased success metrics.
