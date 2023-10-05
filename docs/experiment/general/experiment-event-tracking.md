---
title: Experiment Event Tracking
description: An overview of assignment and exposure tracking in Amplitude Experiment.
---

Amplitude Experiment's *end-to-end* platform relies on two events, [assignment](#assignment-events) and [exposure](#exposure-events), and an [experiment user property](#experiment-user-properties) per experiment, to enable experiment analysis, monitoring, and debugging.

Using the **Amplitude defined exposure or assignment events** as your experiment's exposure event **is recommended** to ensure the correct [experiment user property](#experiment-user-properties) is set when the exposure is ingested. Custom exposure events may be ingested before the experiment user property is set, which would not count in experiment analysis.

<table>
    <tbody>
        <tr>
            <th style="width: 50%;">Assignment</th>
            <th style="width: 50%;">Exposure</th>
        </tr>
        <tr>
            <td>
                <ul>
                <li>Tracked when a user is <b>assigned</b>, as a result of a remote evaluation  (<code>fetch()</code>) or server-side local evaluation (<code>evaluate()</code>).</li>
                <li>Contains assignment information for <b>one or more</b> flags and experiments.</li>
                <li>Useful for monitoring and debugging, or as an exposure heuristic for server-side experiments.</li>
                <li>Sets experiment user properties for all evaluated flags or experiments.</li>
                </ul>
            </td>
            <td>
                <ul>
                <li>Tracked when the user is <b>exposed</b> to a variant, generally on the client-side when a variant is accessed from the SDK (<code>variant()</code>).</li>
                <li>Contains exposure information for a <b>single</b> flag or experiment.</li>
                <li>Used as the exposure event for client-side experiments.</li>
                <li>Sets the experiment user property for the exposed flags or experiment</li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>

!!!info "Event Volume Billing & Property Limits"
    If you've purchased Amplitude Experiment's *End-to-end* feature flagging and experimentation solution, Amplitude defined [assignment](#assignment-events) and [exposure](#exposure-events) **don't count** toward your organization's event volume billing. Further, experiment user properties and event properties don't count towards your project property limits.

    If you have purchased *Experiment Results* or haven't purchased Experiment then all events are billed in full.

## Experiment user properties

Experiment utilizes a user property per flag and experiment which is set or unset on both assignment and exposure events. This user property is used to determine which variant the user is in for experiment analysis purposes.

The format of the user property is, `[Experiment] <flag_key>` and the value is the variant key that the user was assigned or exposed to. You may use this user property in queries for non-experiment events which are ingested after the user property is set to segment based on the flag or experiment variant.

## Assignment events

Assignment events are tracked by Amplitude's evaluation servers or SDKs as a result of [remote evaluation](./evaluation/remote-evaluation.md), or [local evaluation](./evaluation/local-evaluation.md) using a server-side SDK configured for [automatic assignment tracking](#automatic-assignment-tracking). Assignment events are used as a **heuristic exposure event** for server-side experiments, but are also useful for **monitoring** a flag or experiment while active, and **debugging** any issues. When running server-side experiments where client-side exposure tracking isn't feasible, choose the Amplitude Assignment event as the exposure event when setting up your experiment.

You shouldn't need to track assignment events manually.

!!!warning "User property inheritance"
    Assignment events inherit all non-experiment user properties from the current user state in Amplitude at the time of event ingestion. In short, **the user properties on the assignment event aren't guaranteed to be the same as the properties used in evaluation.** For example, If an assignment event is the first event ingested for a user, the event won't contain any non-experiment user properties, even if user properties were explicitly included in the evaluation.

### Assignment event definition

The assignment event, `[Experiment] Assignment`, contains an event property, `[Experiment] <flag_key>.variant`, for each evaluated flag or experiment, where the property value is the assigned variant key or `off` if no variant was assigned. Additionally, the assignment event sets or unsets [experiment user properties](#experiment-user-properties) for each assigned, or unassigned variant respectively. The assignment event contains other event properties like `[Experiment] Environment Name` and `[Experiment] <flag_key>.details` which are useful for internal debugging.

!!!json "Example event JSON"
    This is an example assignment event for a user, `123456789`, who was evaluated for one flag `my-flag` and one experiment `my-experiment`.
    ```json
    {
        "user_id": "123456789",
        "event_type": "[Experiment] Assignment",
        "event_properties": {
            "[Experiment] my-flag.variant": "off",
            "[Experiment] my-experiment.variant": "treatment"
        },
        "user_properties": {
            "$set": {
                "[Experiment] my-experiment": "treatment"
            },
            "$unset": {
                "[Experiment] my-flag": "-"
            }
        }
    }
    ```

### Automatic assignment tracking

Automatic assignment tracking for [remote evaluation](./evaluation/remote-evaluation.md) is supported out-of-the-box. Remote evaluation requests that miss the CDN cache, and which contain a valid user or device ID, will trigger an assignment event to be tracked asynchronously after evaluation.

For server-side [local evaluation](./evaluation/local-evaluation.md), you may configure the local evaluation SDK on initialization to track assignment events on `evaluate()`. Assignment events sent by server-side local evaluation SDKs are deduplicated for each user using an `insert_id` containing the user ID, device ID, hash of a canonicalized list of assigned flags and variants, and finally the date stamp. In other words, you should expect one Assignment per evaluated user, per unique evaluation result, per day.

| SDK | Minimum Version |
| --- | --- |
| [:node:  Node.js](../sdks/nodejs-sdk.md) | `1.7.4+` |
| [:ruby:  Ruby](../sdks/ruby-sdk.md) | `1.2.2+` |
| [:java:  JVM](../sdks/jvm-sdk.md) | `1.2.1+` |
| [:golang:  Go](../sdks/go-sdk.md) | `1.2.2+` |
| [:python:  Python](../sdks/python-sdk.md) | `1.2.3+` |

## Exposure events

An exposure event is a [strictly defined](#exposure-event) analytics event sent to Amplitude to inform Amplitude Experiment that a user was exposed to a variant of an [experiment or feature flag](./data-model.md#flags-and-experiments). Exposure events contain the **flag key** and the **variant** of the flag or experiment that the user has been exposed to in the event's event properties.

When Amplitude ingests an [exposure event](#exposure-event), it uses the flag key and variant to **set or unset user properties** on the user associated with the event. Setting user properties is essential for experiment analysis queries on primary and secondary success metrics.

### Exposure event definition

The exposure event is simple enough to send through any analytics implementation or customer data platform without needing to manipulate user properties directly. When Amplitude ingests an **`$exposure`** event, the requisite user properties are automatically set or unset for 100% accuracy and confidence.

| Event Type | <div class='big-column'>Event Property</div> | Requirement | Description |
| --- | --- | --- | --- |
| **`$exposure`** | `flag_key` | Required | The flag or experiment key which the user is being exposed to. |
| | `variant` | Optional | The variant for the flag or experiment that the user has been exposed to. If `null` or missing, the user property for the flag/experiment is unset, and the users is no longer a part of the experiment. |
| | `experiment_key` | Optional | The key of the experiment that the user was exposed to. The experiment key is used to differentiate between two [runs of an experiment on the same flag key](../guides/troubleshooting/restarting-experiments.md). |

!!!json "Example event JSON"
    This is an example exposure event for a user, `123456789`, who was exposed to the `treatment` variant of the experiment, `my-experiment`.
    ```json
    {
        "user_id": "123456789",
        "event_type": "$exposure",
        "event_properties": {
            "flag_key": "my-experiment",
            "variant": "treatment"
        },
        "user_properties": {
            "$set": {
                "[Experiment] my-experiment": "treatment"
            },
        }
    }
    ```

### Automatic exposure tracking

Client-side Experiment SDKs support automatic exposure tracking through an exposure tracking provider implementation. Without an integration or custom implementation, exposure events aren't tracked automatically.

<!--vale off-->
| <div class='big-column'>SDK Integrations</div> | Minimum Version |
| --- | --- |
| [:javascript-color: JavaScript SDK](../sdks/javascript-sdk.md#integrations) | `1.4.1+` |
| [:android: Android SDK](../sdks/android-sdk.md#integrations) | `1.5.1+` |
| [:apple-black: iOS SDK](../sdks/ios-sdk.md#integrations) | `1.6.0+` |
| [:react: React Native](../sdks/react-native-sdk.md#integrations) | `0.6.0+` |
<!-- vale on-->

### Exposure tracking example

--8<-- "includes/experiment-interactive-exposure-tracking-table.md"
