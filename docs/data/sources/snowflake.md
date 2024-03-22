---
title: Import Snowflake Data
description: Use Amplitude's Snowflake integration to ingest Snowflake data directly into your Amplitude project. 
---

With Amplitude's Snowflake integration, you can ingest Snowflake data directly into your Amplitude project. This article walks you through the steps needed to make that happen.

--8<-- "includes/editions-all-editions.md"

!!!note "Other Amplitude + Snowflake integrations"

    This integration imports Snowflake data into Amplitude. Amplitude offers other integrations with Snowflake: 

    - [Export Amplitude Data to Snowflake](/data/destinations/snowflake)

## Considerations

- Depending on your company's network policy, you may need add these IP addresses to your allowlist in order for Amplitude's servers to access your Snowflake instance:

    - Amplitude US IP addresses:
        - 52.33.3.219
        - 35.162.216.242
        - 52.27.10.221 
    - Amplitude EU IP addresses:
        - 3.124.22.25
        - 18.157.59.125
        - 18.192.47.195

## Limits

- Maximum running time for a single Snowflake SQL query is 12 hours.

!!!warning "User and Group Properties Sync"

      Amplitude's Data Warehouse Import sometimes processes events in parallel, so time-ordered syncing of user and group properties on events is not guaranteed in the same way as submitting events directly to the Identify and Group Identify APIs. 

## Modeling methods

Amplitude's Snowflake Data Import supports two methods for importing data from Snowflake, Table Selection UI and Custom SQL Query.

|                   | Table Selection UI                                                                                                                                                                                 | Custom SQL Query                                                                                                                |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Import data types | Event, User property, Group Property                                                                                                                                                               | Event, User property, Group Property                                                                                            |
| Import strategy   | Change-based                                                                                                                                                                                       | Time-based, Full Sync (only for group and user properties)                                                                      |
| When to use       | Recommended for most use cases, user-friendly, minimal SQL knowledge required. <br/>Limited data source selection functionality, consider creating Snowflake VIEW (see Prerequisites for details). | Use when data selection requires customization, even though it may lead to data discrepancies and higher costs if misconfigured |

### Table selection UI

The Table Selection UI modeling method supports Change Data Capture (CDC). This method identifies and captures changes made to data in a database and delivers those changes in real time to a downstream process or system.

For the Snowflake source in Amplitude, Table Selection UI uses CDC mechanisms available in Snowflake, [Time Travel](https://docs.snowflake.com/en/user-guide/data-time-travel) and [CHANGES](https://docs.snowflake.com/en/sql-reference/constructs/changes) clause, to identify changes made in the data source since the last successfully completed import job.

#### Prerequisites and considerations

- If a data source is represented as a complex SQL SELECT statement (for instance, with a JOIN clause), create a VIEW in your Snowflake account that wraps the data source to use it with a change-based import strategy.
- Enable change tracking for the source table or view. See [Enabling Change Tracking on Views and Underlying Tables Snowflake](https://docs.snowflake.com/en/user-guide/streams-manage.html#label-enabling-change-tracking-views) for more information. 
- `DATA_RETENTION_TIME_IN_DAYS` must be greater than or equal to `1`, but Amplitude recommends at least `7` days. Otherwise, the change-based import fails. For more details, see [Time Travel](https://docs.snowflake.com/en/user-guide/data-time-travel) in Snowflake's documentation. Setting `DATA_RETENTION_TIME_IN_DAYS` to  `0` disables the change tracking, and causes the connection to become unrecoverable. If this happens, recreate the source.
- [Data field](#data-fields) requirements also apply.
- (Optional, recommended) Ensure the data to be imported has a unique and immutable `insert_id` for each row to prevent data duplication if there are any unexpected issues. More about Amplitude deduplication and `insert_id` is [Event Deduplication](https://www.docs.developers.amplitude.com/analytics/apis/http-v2-api/#event-deduplication).
- If you disable change tracking in Snowflake, or disconnect the Amplitude source for a period longer than the value of `DATA_RETENTION_TIME_IN_DAYS`, Amplitude loses ability to track historical changes. In this case, recreate the connection. To avoid duplicate events, ensure all events have an `insert_id` set, and recreate the connection within seven days.
- The initial import job transfers all data from the source. Subsequent jobs import the differences from the last successful import.
- Snowflake [`CHANGES`](https://docs.snowflake.com/en/sql-reference/constructs/changes#usage-notes) limitations apply.

### Custom SQL query

The Custom SQL query supports time-based import of events, user properties, and group properties, and full syncs of user properties and group properties.

For Time-based import, Amplitude requires that you use a monotonically increasing timestamp value. This value should show when the record was loaded into the source table the SQL configuration is querying from. The warehouse import tool brings data into Amplitude by continually updating the maximum value of the column referenced in the *Timestamp Column Name* input within the Import Config UI with each subsequent import.

!!!example

    Upon first import, Amplitude imports all the data returned from the query configured in the Import Config. Amplitude saves a reference of the maximum timestamp referenced in the *Timestamp Column Name*: `timestamp_1`. Upon subsequent import, Amplitude imports all data from the previously saved timestamp (`timestamp_1`), to what's now the new maximum timestamp (`timestamp_2`). Then after that import, Amplitude saves `timestamp_2` as the new maximum timestamp.

## Add Snowflake as a source

To add Snowflake as a data source in your Amplitude project, follow these steps:

1. In Amplitude Data, click **Catalog** and select the **Sources** tab.
2. In the Warehouse Sources section, click **Snowflake**.
3. Enter the required credentials for the Snowflake instance you want to connect:
      - **Account**: Snowflake account name. Case sensitive. This is the first part of your Snowflake URL, before `snowflakecomputing.com`. Don't include             ".snowflakecomputing.com" in your account name. 
      - **Database**: Name of the database where Amplitude can find the data.
      - **Warehouse**: Used by Amplitude to execute SQL.
      - **Username**: Used by Amplitude for authentication.
      - **Password**: Used by Amplitude for authentication.
4. Copy the autogenerated SQL query and run it in Snowflake to give Amplitude the proper permissions. 
5. After running the query, click **Next** to test the connection.
6. After the test is successful, click **Next** again to move on to the data selection stage.
7. Choose the modeling method, [Table Selection UI](#table-selection-ui-settings) or [Custom SQL Query](#custom-sql-query-settings).

### Table Selection UI settings

Configure the modeling method:

- **Data source**: Choose a table or view from the left panel.

- **Data type**: Select if the table maps to  event, user property, or group property data.

- **Frequency**: Select the interval with which Amplitude should check for changes in the Snowflake table.

Map the required and custom fields: Setup name mapping between columns in the Snowflake data source and data field name that Amplitude requires. For more information, see [Data fields](#data-fields) below.

When complete, click **Test Mapping** to verify the correct data appears under the right property in Amplitude.

### Custom SQL query settings

Choose your configuration options: 

- **Type of data**: This tells Amplitude whether you're ingesting event data, user property data, or group property data.
- **Type of import:**
  - **Full Sync**: Amplitude periodically ingests the entire dataset, regardless of whether that data has already been imported. This is good for data sets where the row data changes over time, but there is no easy way to tell which rows have changed. Otherwise, the more efficient option would be a time-based import. This option isn't supported for ingesting event data.
  - **Time-based**: Amplitude periodically ingests the most recent rows in the data, as determined by the provided *Timestamp* column. The first import brings in all available data, and later ingests any data with timestamps after the maximum timestamp seen during the last import job. To use this, include the timestamp of the data load into Snowflake. For more information on how this works, see [the time-based import](#time-based-import) section.
- **Frequency**: Choose from several scheduling options ranging from five minutes to one month. With the one month option, Amplitude ingests data on the first of the month.
- **SQL query**: This is the code for the query Amplitude uses to decide which data is ingested.

Finish the configuration:

1. After you've set your configuration options, click **Test SQL** to see how the data is coming through from your Snowflake instance. Errors appear on this screen.
2. If there are no errors, click **Finish**. 

Amplitude displays a notification indicating you enable the new Snowflake source and redirects you to the Sources listing page.

If you have any issues or questions while following this flow, contact the Amplitude team.

## Migrate from Custom SQL to Table Selection UI

To change the modeling method of your Snowflake source:

1. (Optional, recommended). Ensure the data you plan to import has a unique and immutable `insert_id` in each row to prevent data duplication. For more information, see [Data deduplication](/analytics/apis/http-v2-api/#event-deduplication).
2. If the source uses complex SQL, including `JOIN` and `WHERE` clauses:
      1. Create a [`VIEW`](https://docs.snowflake.com/en/user-guide/views-introduction) in your Snowflake account that wraps the data source.
      2. Enable [Change Tracking](https://docs.snowflake.com/en/user-guide/streams-manage.html#label-enabling-change-tracking-views) on the new view.
      3. Update the current Snowflake SQL import configuration to use the newly created view. Record the time of the update.
      4. Ensure `Data synced as of` is greater than the time recorded in the previous step to prevent potential data discrepancy and failure to identify the data drift after the latest completed import job.
3. Enable [Change Tracking](https://docs.snowflake.com/en/user-guide/streams-manage.html#label-enabling-change-tracking-views) on the source table or view, if you haven't done so. Record the time of the update.
4. Ensure the existing connection has `Data synced as of` (presented on the source detail page) on or after `October 1, 2023, 12:00 AM UTC`. If it doesn't, either re-enable the connection and wait for `Data synced as of` to advance or consider creating a new import connection. Otherwise, Amplitude imports all data from the current source, which may cause data duplication.
5. Ensure `Data synced as of` is greater than the time recorded in the step 3 to prevent potential data discrepancy and failure to identify the data drift after the latest completed import job.
6. Disable the source from the **Manage Import Settings** dialog. If the source has a status of In-Progress, wait for the job to complete and the status changes to Disabled.
7. Navigate to the **Edit Import Config** tab and click **Convert To Table Select Import**.
8. Re-enable the source.
9. Monitor incoming data for one day to ensure the import works as expected.

### Roll back to a Custom SQL connection

To revert to a Custom SQL connection from an already migrated source, open the source configuration and click **Revert to SQL Query Import**.

!!!info "Snowflake data sources"
    When you roll back from the Table Selection UI to Custom SQL connection in the, use the same data source (table or view) in Snowflake to avoid inconsistencies.

## Data fields

Include the mandatory fields for the data type when you create the SQL query. These tables outline the mandatory and optional fields for each data type. Find a list of other supported fields for events in the [HTTP V2 API documentation](/analytics/apis/http-v2-api/#keys-for-the-event-argument) and  for user properties in the [Identify API documentation](/analytics/apis/identify-api/#identification-parameter-keys). Add any columns not in those lists to either `event_properties` or `user_properties`, otherwise it's ignored. 

### Events
<!-- vale off-->
| Column name (must be lowercase) | Mandatory                           | Column data type                     | Example                                       |
| ------------------------------- | ----------------------------------- | ------------------------------------ | --------------------------------------------- |
| `user_id`                       | Yes, unless `device_id` is used     | VARCHAR                              | datamonster@gmail.com                         |
| `device_id`                     | Yes, unless `user_id` is used       | VARCHAR                              | C8F9E604-F01A-4BD9                            |
| `event_type`                    | Yes                                 | VARCHAR                              | watch_tutorial                                |
| `time`                          | Yes                                 | Milliseconds since epoch (Timestamp) | 1396381378123                                 |
| `event_properties`              | Yes                                 | VARIANT (JSON Object)                | {"source":"notification", "server":"host-us"} |
| `user_properties`               | No                                  | VARIANT (JSON Object)                | {"city":"chicago", "gender":"female"}         |
| `update_time_column`            | No (Yes if using time based import) | TIMESTAMP_NTZ                        | 2013-04-05 01:02:03.000                       |

Find other supported fields can in the [HTTP V2 API documentation](/analytics/apis/http-v2-api/#upload-request-headers).

### User properties

| Column name (must be lowercase) | Mandatory                           | Column data type      | Example                               |
| ------------------------------- | ----------------------------------- | --------------------- | ------------------------------------- |
| `user_id`                       | Yes                                 | VARCHAR               | datamonster@gmail.com                 |
| `user_properties`               | Yes                                 | VARIANT (JSON Object) | {"city":"chicago", "gender":"female"} |
| `update_time_column`            | No (Yes if using time based import) | TIMESTAMP_NTZ         | 2013-04-05 01:02:03.000               |

Find other supported fields in the [Identify API documentation](/analytics/apis/identify-api/#identification-parameter-keys).

<!--vale on-->
### Group properties

| Column name (must be lowercase) | Mandatory                           | Column data type      | Example                                                |
| ------------------------------- | ----------------------------------- | --------------------- | ------------------------------------------------------ |
| `groups`                        | Yes                                 | VARIANT (JSON Object) | {"company":"amplitude", "team":["marketing", "sales"]} |
| `group_properties`              | Yes                                 | VARIANT (JSON Object) | {"location":"seattle", "active":"true"}                |
| `update_time_column`            | No (Yes if using time based import) | TIMESTAMP_NTZ         | 2013-04-05 01:02:03.000                                |

Each group property in `group_properties` would apply to every group in `groups`.

### Profile properties

| Column name (must be lowercase) | Mandatory | Column data type             | Example                  |
| ------------------------------- | --------- | ---------------------------- | ------------------------ |
| `user_id`                       | Yes       | VARCHAR                      | "user123"                |
| `property_name_1`               | Yes       | (key value) VARCHAR: VARCHAR | "Title": "Data Engineer" |
| `property_name_1`               | Yes       | (key value) VARCHAR: VARCHAR | "City": "San Francisco"  |

Amplitude supports profile properties for known users. A `user_id` value must join each profile property.

## SQL query examples

To make the data selection step a bit easier, here are few example SQL snippets to get you started.

### Event data example

```sql
SELECT
    EVENT_TYPE_COLUMN AS "event_type",
    EVENT_PROPERTIES_VARIANT_COLUMN AS "event_properties",
    TIME_EPOCH_MS_COLUMN AS "time",
    USER_ID_COLUMN AS "user_id",
    USER_PROPERTIES_VARIANT_COLUMN AS "user_properties"
FROM DATABASE_NAME.SCHEMA_NAME.TABLE_OR_VIEW_NAME
```

### User property example

```sql
SELECT
    USER_ID_COLUMN AS "user_id",
    USER_PROPERTIES_VARIANT_COLUMN AS "user_properties"
FROM DATABASE_NAME.SCHEMA_NAME.TABLE_OR_VIEW_NAME
```

### Group property example

```sql
SELECT
    GROUPS_OBJ AS "groups",
    GROUP_PROPS_OBJ AS "group_properties"
FROM DATABASE_NAME.SCHEMA_NAME.TABLE_OR_VIEW_NAME
```

### Common snippets

Creating a JSON Object:

`OBJECT_CONSTRUCT('city', CITY, 'state', STATE) as "user_properties"`

Converting timestamp column to milliseconds:

`DATE_PART('EPOCH_MILLISECOND', TIMESTAMP_COLUMN) as "time"`

Converting milliseconds to TIMESTAMP_NTZ format needed for time-based import. This example uses the `scale` argument set to `3` to convert to milliseconds. See the [Snowflake documentation](https://docs.snowflake.com/en/sql-reference/functions/to_timestamp.html) for more details.

`TO_TIMESTAMP_NTZ(TIME_COLUMN_IN_MILLIS, 3) as "update_time_column"`

Converting a timestamp column with a timezone to TIMESTAMP_NTZ format needed for time-based import.

`TO_TIMESTAMP_NTZ(CONVERT_TIMEZONE('UTC', TIMESTAMP_TZ_COLUMN)) as "update_time_column"`
