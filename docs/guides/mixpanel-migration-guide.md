---
title: Migrating from Mixpanel to Amplitude
description: This document explains the step-by-step process of migrating from Mixpanel to Amplitude.
---

The Amplitude Professional Services team has compiled this Mixpanel to Amplitude Implementation Guide to help you implement Amplitude and start getting insights right out of the gate.

With the goal of driving tangible conversion, retention, and product outcomes, this guide provides you with recommended use cases and business questions, a proposed taxonomy, and example charts that can be used for your implementation.

!!! info "Customer case study"
    Learn about how Whisk migrated from Mixpanel to Amplitude on the [Amplitude Blog](https://amplitude.com/blog/whisk-product-growth).

## Implementation process

The steps below provide a roadmap to the process of migrating to Amplitude.

### Set product metrics

* Before you dive into the data, ensure all stakeholders and team members agree on what you want to get out of the data. What use cases would the team like to focus on?
* See list of suggested Mixpanel to Amplitude use cases and business questions/metrics down below.

### Design a data taxonomy

* Your data taxonomy should have events and properties that can answer your core metrics.
* See the [Data Taxonomy Playbook](https://help.amplitude.com/hc/en-us/articles/115000465251-Data-Taxonomy-Playbook) and the sample taxonomy below for best practices.

### Instrument your taxonomy

You can send data to Amplitude client-side, server-side, or through a third party. For more information, see [Getting Started with Amplitude](/getting-started).

## Suggested use cases and business questions

Amplitude provides a workbook with the most common use cases from other Amplitude customers who've migrated from Mixpanel. For more information, see the [Mixpanel to Amplitude Migration Workbook](https://docs.google.com/spreadsheets/d/1lsZa6uZmcUmJdq-_sr5JawckMPiiQDCCzl_ytSYccNg/edit#gid1755544121)

### Common use cases by industry

=== "B2B"
    **Goal**: Understand product engagement.

    **Key points:**

    * See how your users convert through critical funnels: acquisition (free trial, sales, partner, POC), onboarding, activation, workflow, cross-sell/upsell funnels.
    * Target the right customers at the right time to move them through a critical funnel.
    * Find patterns in the way your customers move through key milestones (acquisition, onboarding, activation, renewal).
    * Understand different customer segments’ use and adoption to define key personas based on use cases and needs.
    * Optimize your product experience to target different customers personas needs and make them more successful.
=== "Fintech"
    **Goal**: Understand what makes your users purchase.

    **Key questions:**

    * What's the efficiency of marketing channels?
    * How many users complete sign-up and money transfer in one session?
    * How do users engage with product features?
    * What impacts user retention?
    * What's the % of users that have used accounts with > 2 currencies in the last 30 days?
    * How much revenue do we get from a customer?
    * What impacts referrals?
=== "E-commerce"
    **Goal:** Understand revenue and conversion drivers.

    **Key questions:**

    * How often do users look at products?
    * What is our purchase conversion rate?
    * And what is the falloff in each step? (rates of- click to category page, click to product, add to cart, view cart, start checkout, order conversion)
    * What features do users interact with that lead to conversions?
    * What are the drivers that lead from user registration to first purchase?
    * How many purchases include more than one item?
    * Does the ATC (add to cart) decreased by user or device type?
=== "Streaming media"
    **Goal:** Understand the acquisition and subscription drivers of your users' engagement with your product.

    **Key questions:**

    * What's the total length of time from content consumption in the Free Trial for churned trials vs. subscription converters?
    * What activities are common between users who convert vs don't convert after the Free Trial expires?
    * How do users interact with our site and how do they consume content?
    * What is the percentage of content consumed per genre?
    * What brings users back?

## Suggested Mixpanel to Amplitude taxonomy

Amplitude provides the following to help your migration:

* A suggested [Mixpanel to Amplitude Taxonomy](https://docs.google.com/spreadsheets/d/1lsZa6uZmcUmJdq-_sr5JawckMPiiQDCCzl_ytSYccNg/edit#gid434510064)for tracking a minimum set of Events, Event Properties, User Properties.
* A [sample notebook](https://analytics.amplitude.com/share/db099a8a568447fb931b4f476fc0ecf6) containing helpful analysis and charts to get started.

## Recommended instrumentation

Send live event data from your website to Amplitude and/or migrate your existing historical data from Mixpanel to Amplitude.

### Send live event data from your product to Amplitude

Amplitude's recommendation depends on the method you use to track events. 

For client-side event tracking:

* [Amplitude’s SDK Catalog](https://www.docs.developers.amplitude.com/data/sdks/sdk-overview/). Use your existing instrumentation method and reference Amplitude mapping to replace instrumentation.

For server-side event tracking:

* [Amplitude's HTTP API](https://www.docs.developers.amplitude.com/analytics/apis/http-v2-api/?hhttp)

### Map existing Mixpanel methods to Amplitude

Amplitude offers several methods that map to Mixpanel methods.

#### Event tracking

* Mixpanel track events with the `'mixpanel.track()'` method, which takes an event name and sets of properties.
* Amplitude tracks events with the `'amplitude.getinstance().logEvent()'` method. This also takes an event name and set of properties as a JSON object.

#### Property tracking

Super properties in Mixpanel are properties that attach to all subsequent events. Amplitude's User Properties function similarly in that once set, these properties attach to all subsequent events that Amplitude ingests. 

* In Mixpanel, super properties are set with the `'mixpanel.register()'` method.
* In Amplitude, update user properties with the `'amplitude.identify()'` method.

#### User identification

* Mixpanel uses a combination of `distinct_id` (a randomly generated identifier on a specific platform) and `user_id` (explicitly set by the instrumenting teams) to identify a user with the `'mixpanel.identify()'` method.
* Amplitude uses a combination of `device_id` (a randomly generated id on a specific platform) and `user_id` (explicitly set by the instrumenting teams) to identify a user with the `'amplitude.identify()'` method.

For more on how Amplitude resolves user identifies, see [Track unique users](https://help.amplitude.com/hc/en-us/articles/115003135607-Track-unique-users-in-Amplitude#h_7cf7c47f-ec71-4e15-8c47-a2bda5d84186)

### Migrate existing historical data from Mixpanel to Amplitude

Amplitude offers a few options to migrate your historical data from Mixpanel to Amplitude:

* [Amplitude Batch Event Upload API](https://www.docs.developers.amplitude.com/analytics/apis/batch-event-upload-api/)
  * Export your data directly out of Mixpanel via [API](https://developer.mixpanel.com/reference/raw-data-export-api) and upload into Amplitude with the Batch API.
  * If you host your data in a another external source, you can also use the batch endpoint to upload data into Amplitude.
* Professional Services
  * For custom services lead by the Amplitude Professional Services team, contact your Amplitude account manager.

### Data privacy considerations

Along with the ingestion methods above, here are some features or areas to consider when managing your customer data:

* [Time to Live (TTL)](https://www.docs.developers.amplitude.com/data/ttl-configuration/?htime+live) - feature within Amplitude to control how long event data lives in your Amplitude instance
* [How to manage Opt-Outs](https://www.docs.developers.amplitude.com/data/sdks/typescript-browser/?hopt+out#opt-users-out-of-tracking) - SDK settings to allow your website visitors to disable activity tracking on your website
* [IP Address](https://www.docs.developers.amplitude.com/analytics/apis/batch-event-upload-api/?hip+address#parameters) - Amplitude captures IP address and location-based details by default in with client-side tracking. For information about disabling this tracking, see [Browser SDK | Optional Tracking](https://www.docs.developers.amplitude.com/data/sdks/browser-2/#optional-tracking) 
* [User Privacy API](https://www.docs.developers.amplitude.com/analytics/apis/user-privacy-api/) - API to delete all data for a set of known Amplitude IDs or User IDs.
* [Amplitude’s Stance on Security & Privacy](https://amplitude.com/amplitude-security-and-privacy)

## GDPR INFORMATION

Amplitude is fully GDPR compliant.

For more information about compliance, see [Security and Privacy](https://amplitude.com/amplitude-security-and-privacy).

Amplitude maintains a [user privacy API](https://www.docs.developers.amplitude.com/analytics/apis/user-privacy-api/) that allows you to service end user data deletion requests.

## Feedback or questions

For any feedback or questions on this implementation guide, submit them [here](https://forms.gle/EMh9JeNs1iNCQzx67).