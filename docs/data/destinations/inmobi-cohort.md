---
title: Send Cohorts to InMobi
description: Sync cohorts from Amplitude to InMobi
---

!!!Tip

    This guide provides you with an overview of how to sync cohorts from Amplitude to InMobi via Amazon S3. i.e. You will be syncing cohorts to a specified Amazon S3 bucket that InMobi manages. InMobi will read your Amazon S3 file and ingest the data on a daily basis.

[InMobi](https://www.inmobi.com/) is a complete mobile marketing and advertising platform for advertisers and publishers.

## Considerations

- This integration isn't a native integration from Amplitude to InMobi.
- This integration is available for customers who have paid plans with Amplitude.
- You must enable this integration in each Amplitude project you want to use it in.
- This integration requires a paid InMobi plan.
- InMobi expects an identifier that matches the InMobi Custom User ID field. This means the `user_id` or user property you select in Amplitude must contain the same identifier as the one tracked in InMobi.

## Setup

### InMobi setup

1. Contact your InMobi CSM team for the following Amazon S3 details as you need this when setting up your Amazon S3 cohort destination in Amplitude.
   - **Bucket Name:** An Amazon bucket name refers to the unique name assigned to a storage container in Amazon.
   - **Bucket Region:** The hosting region of the Amazon S3 bucket.

### Amplitude setup

1. In Amplitude Data, click **Catalog** and select the **Destinations** tab.
2. In the Cohort section, click **Amazon S3 (Cohorts)**.
3. Paste in the details you've obtained from the InMobi team under **Bucket Name**, and **Bucket region**.
4. Enter your **Bucket Path** (which could be your `advertiser_id`), and enter a **Name** for the destination. The name is used to help you identify which S3 bucket destination when syncing a cohort from Amplitude.
5. Optionally, you can also set the following two parameters for your buckets:
   - **Require suffix:** When set, this allows users to append a string at the end of every file exported to S3.
   - **Amplitude User property:** You can select a single user property to sync along with each user as an extra column in each file exported. Select the Amplitude user property. For example, you can use email or device_ID to match users between Amazon S3 and Amplitude.
6. Click **Copy Bucket Policy**.
7. Click **Save** to complete the setup. 
8. Pass the saved bucket policy to your InMobi team so they can add the S3 bucket policy for you. The InMobi team owns the Amazon S3 bucket and must to configure it on your behalf.

## Send a cohort

To sync your first cohort:

1. In Amplitude, open the cohort you want to sync, then click **Sync**.
2. Select **Amazon S3**, then click **Next**.
3. Select the S3 location. This is what you named the bucket when setting up the integration.
4. (Optional). Set the following two optional parameters:
   - **Routing Key:** Enter a string to append to the end of the cohort file name in S3. Amplitude recommends `advertiser_id `as the routing key.
   - **Define User Properties**: Here, you can append a user property to each user exported in this cohort. The user property appears as a column in the exported CSV file.
5. Choose a sync cadence.
6. When finished, click **Sync**.

## Use cases

1. **Targeted Advertising:** Sending specific cohorts from Amplitude, a product analytics platform, to InMobi enables advertisers to target their campaigns more effectively. Cohorts in Amplitude, which are groups of users segmented based on shared behaviors or characteristics, enable more personalized and relevant advertising when exported to InMobi.
2. **Improved User Engagement:** Understanding user behavior through Amplitude allows for the creation of cohorts based on engagement levels, feature usage, or other significant activities. Sending these cohorts to InMobi enables the tailoring of ads to increase user engagement with the product.
3. **Retargeting and Remarketing:** Retargeting and remarketing become possible by utilizing cohorts composed of users who have shown interest in specific products or services but haven't converted. InMobi can retarget these potential customers who are already familiar with the product.
