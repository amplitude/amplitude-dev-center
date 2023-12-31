---
title: Google Ads Event Streaming
description: Amplitude Data's Google Ads Event streaming integration enables you to stream your Amplitude event data straight to Google Ads with just a few clicks.
status: new
---

Amplitude Data's Google Ads integration enables you to stream your Amplitude event data straight to Google Ads with just a few clicks.

## Considerations

- You must enable this integration in each Amplitude project you want to use it in.
- Amplitude sends custom events using Amplitude `event_type` as event name.
- This integration uses Google's [Upload Click Conversions API](https://developers.google.com/google-ads/api/docs/conversions/upload-clicks)).
- This integration requires a Google Click ID (`gclid` or `initial_gclid` to enable Amplitude to create the click conversion and pass that event to Google Ads. Google Ads relies on the `gclid` parameter to track user interactions with ads and attribute conversions back to specific ad clicks. When a user clicks an ad served through Google Ads, Google Ads generates a unique `gclid` value and appends it to the URL. This parameter identifies the click, and links it to subsequent conversions.

## Setup

### Prerequisites

To set up event streaming to Google Ads, you need the following:

- A Google Ads Customer ID
- A Google Ads Conversion Action ID
- A [Google Cloud Service Account](https://cloud.google.com/iam/docs/service-accounts-create)

### Amplitude setup

1. In Amplitude Data, click **Catalog** and select the **Destinations** tab.
2. In the Event Streaming section, click **Google Ads**.
3. Enter a sync name, then click **Create Sync**.

After you create the destination, you must configure the settings.

#### Configure settings

1. On the **Settings** tab, click **Edit**.
2. Enter your [**Google Ads Customer ID**](https://support.google.com/google-ads/answer/1704344?hl=en). A Google Ads Customer ID is a unique identifier assigned to each advertiser or business that uses Google Ads. This ID helps Google track and manage accounts, campaigns, and billing information for advertisers.
3. Enter your [**Google Ads Conversion Action ID**](https://support.google.com/google-ads/thread/105330243?hl=en&sjid=5504033552721490234-EU). This is a unique identifier associated with a specific conversion action in Google Ads, and represents a desired action that you want your visitors or users to take. For example, make a purchase, submit a contact form, or sign up for a newsletter.
4. Enter your [**Base64 encoded Google Cloud Service Account**](https://developers.google.com/google-ads/api/docs/oauth/service-accounts). A Google Cloud Service Account, also known as a Service Account, is a special type of Google account used for server-to-server interactions and authentication within Google Cloud Platform (GCP) services. It's can grant permissions and access to resources like Google Cloud Storage, Google Cloud Compute Engine, or other GCP services. Service Accounts have associated credentials (private keys or OAuth tokens) for authentication when you interact with GCP services programmatically.
5. Configure **Send Events** to send events ingested by Amplitude to Google Ads.
      1. To send events, toggle **Send Events** to **Enabled**.
      2. Expand the **Select and filter events** panel, and select which events to send. Amplitude recommends that you send only the events you need in Google Ads, rather than selecting **All Events**.
6. Save when finished.

#### Enable integration

The final step is enabling the destination. You must enable the destination to start streaming events.

1. Navigate back to the **Settings** tab and click **Edit**.
2. Toggle **Status** from **Disabled** to **Enabled**.
3. Save when finished.

## Use Cases

1. **Accurate Conversion Tracking:** The integration allows you to track conversions more accurately by sending Amplitude event data directly to Google Ads. This ensures that you can attribute conversions to specific user actions and events, helping you understand which aspects of your campaigns are driving results.
2. **Optimizing Campaigns:** With real-time data from Amplitude, you can optimize your Google Ads campaigns based on user behavior and engagement. This means you can make data-driven decisions to allocate your budget to the most effective keywords, ad creatives, and targeting options.
3. **Personalization:** With Amplitude's event data, you can create highly personalized ad campaigns. For example, you can use the data to target users who have completed specific actions on your website or app with tailored ad messages or offers.

## Common issues

### Stuck at the authentication step

If you see `ERR_BLOCKED_BY_CLIENT` in your browser's console, disable your ad blocker, clear your browser's cache and cookies, and try to add the connection again.

### "Error: Invalid Customer ID"

Google Ads requires a **Customer ID** to configure as an integration with Amplitude. For more information, see [Find your Google Ads customer ID](https://support.google.com/google-ads/answer/1704344?hl=en) in Google's documentation.
Keep in mind, **Manager Account Customer ID** is different from the **Customer ID**.

### Insufficient permissions

Amplitude's Google Ads Event Streaming integration requires that your Google Ads Manager account has administrator privileges. This level of permission allows Amplitude to add and remove users from specific user lists in Google Ads.
For Google Ads, Amplitude adds and removes users from a specific user list. As a result, Amplitude needs appropriate permissions to create and delete Google Ads account data.
For more information, see [About access levels in your Google Ads Account
](https://support.google.com/google-ads/answer/9978556) in Google's documentation. 
