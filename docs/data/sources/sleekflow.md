---
title: Import Sleekflow Events
description: Import Sleekflow events into Amplitude
---

[Sleekflow](http://www.Sleekflow.com/) is an Omni-Channel Social Engagement Platform that helps companies and users manage communication channels like WhatsApp, Facebook Messenger, WeChat, Line, and Instagram. Sleekflow handles both inbound messaging management as well as outbound messaging campaigns. This integration lets you sync cohorts from Amplitude to Sleekflow.

Sleekflow's integration sends events from Sleekflow to Amplitude.

!!!note "Other Amplitude + Sleekflow integrations"

    This integration imports Sleekflow data into Amplitude. Amplitude offers other integrations with Sleekflow: 

    - [Send Amplitude Cohorts to Sleekflow](/data/destinations/sleekflow-cohort)

## Considerations

- You need a Sleekflow account and an Amplitude plan.
- This integration supports these identifiers:
    - User ID: Email address
    - Connection type: Server

## Setup

1. In Sleekflow, navigate to the **Automation** tab.
2. Click **Create new rule**.
3. Click **Customized a rule**.
4. Click **Incoming messages**.
5. Under **Conditions**, click on **Add condition**, and select **Phone Number**.
6. Under **contains**, select "**is known as**".
7. Under **Actions**, click on the big X near the right.
8. Under **Add action**, select "Send webhook".
9. Click **Publish rule**.
10. In the blank, fill in: https://sleekflow-amplitude-f375paklaq-uc.a.run.app/receive-message?apiKey={api_key}.
11. Repeat steps 2-8. In step 4, select **Outgoing Messages**.
12. In the same blank as step 9, fill in: https://sleekflow-amplitude-f375paklaq-uc.a.run.app/send-message?apiKey={api_key}.
13. Click **Publish rule**.

## Use cases

1. **User Behavior Analysis:** Track how users engage with your brand or business through different messaging platforms. Monitor when users open messages, click links, make inquiries, or complete transactions. This data helps you understand user behavior and preferences.
2. **Conversion Tracking:** When you send events related to conversions (like completing a purchase, signing up for a newsletter, or requesting more information) to Amplitude, you can measure the effectiveness of your messaging campaigns on different platforms. This information can help you optimize your marketing strategies.
3. **User Segmentation:** Amplitude allows you to create user segments based on specific actions and behaviors. Use Sleekflow's event data to create custom segments of users who've interacted with your business in particular ways on different channels. Use this segmentation for targeted messaging and marketing efforts.
