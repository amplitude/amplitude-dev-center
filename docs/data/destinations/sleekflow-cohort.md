---
title: Send Cohorts to Sleekflow
description: Sync cohorts from Amplitude to Sleekflow
status: new
---

!!!beta

    This integration is in beta and is in active development. The integration is maintained by Master Concept Group. For support or questions about the integration, contact the Master Concept team directly at [digital_lab@hkmci.com](mailto:digital_lab@hkmci.com).

[Sleekflow](https://sleekflow.io/) is an Omni-Channel Social Engagement Platform that helps companies and users manage communication channels like WhatsApp, Facebook Messenger, WeChat, Line, and Instagram. Sleekflow handles both inbound messaging management as well as outbound messaging campaigns. This integration lets you sync cohorts from Amplitude to Sleekflow. 

## Considerations

- This integration is available by request. Email [digital_lab@hkmci.com](mailto:digital_lab@hkmci.com) to request access.
- You must enable this integration in each Amplitude project you want to use it in.
- You must have a paid Sleekflow plan to enable this integration.
- Sleekflow requires at least an email address or a phone number as the unique contact identifier. The phone number must include a country code (the `+` in front of the country code is optional). This means the `user_ID` or user property you select in Amplitude must contain an email address or phone number.

## Setup

### Sleekflow setup

1. Log in to Sleekflow.
2. Navigate to Channels > Integrations > More customizable extensions.
3. Click **Connect** next to **API**.
4. Copy the Sleekflow API Key.

### Amplitude setup

1. In Amplitude Data, click **Catalog** and select the **Destinations** tab.
2. In the Cohort section, click **Sleekflow**.
3. Click **Add another destination**.
4. Enter **Name** and paste in the **API** key you copied from **Sleekflow**.
5. Map the **Amplitude fields** (which must contain an email address or phone number) to the **Sleekflow** fields.
6. Save when finished.

## Send a cohort

To sync your first cohort:

1. In Amplitude, open the cohort you want to sync, then click **Sync**.
2. Select **Sleekflow**, then click **Next**.
3. Choose the account you want to sync to.
4. Choose the sync cadence.
5. When finished, save your work.

### Use cases

1. **Personalized Push Notifications:**  When you integrate Amplitude cohorts with Sleekflow, you can send personalized messages to different user segments based on their behavior and preferences. For example, you can create a cohort of users who have abandoned their shopping carts then send them targeted reminders or incentives through WhatsApp, Facebook Messenger, or other communication channels that Sleekflow supports. 
2. **Re-Engagement Campaigns:** Use Amplitude cohorts to identify users who haven't interacted with your brand for a certain period. Sleekflow can send re-engagement messages to these cohorts through supported social media channels, encouraging them to revisit your platform and make a purchase.
3. **Product Recommendations:** Use Amplitude's user behavior data to create cohorts of users with similar product interests. Then, use Sleekflow to send product recommendations and promotions to these cohorts through channels like Instagram or Facebook Messenger, increasing the likelihood of conversions.
