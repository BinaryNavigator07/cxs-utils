---
title: Understanding Datapoints
---

# Understanding Datapoints in the Semantic Schema

## Introduction

A **Datapoint** within the Semantic Event Schema represents a specific, granular piece of information, typically a measurement or an attribute, that provides detailed context about a Semantic Event or an Entity at a particular point in time or over a defined period. Think of datapoints as the lowest-level, quantifiable or descriptive details that enrich the understanding of what happened, what an entity is like, or how a metric is changing.

The primary purpose of Datapoints is to offer precise, low-level details necessary for quantification, detailed description, timeseries analysis, and in-depth analytics. They are the raw facts that, when aggregated and analyzed, lead to meaningful insights and a deeper understanding of complex systems.

## Key Characteristics of Datapoints

While the detailed schema is covered below, datapoints generally possess these characteristics:

- **Name/Key**: A unique identifier or label for the datapoint (e.g., `page_load_time`, `product_price`, `error_code`). This describes what the datapoint represents.
- **Value**: The actual recorded data for that datapoint (e.g., `150` for `page_load_time_ms`, `49.99` for `product_price_usd`).
- **Data Type**: The nature of the value, such as number, string, boolean, array, or object. This dictates how the datapoint can be processed and analyzed.
- **Units (Recommended for Numerical Data)**: For numerical data, specifying units is crucial for correct interpretation (e.g., `ms` for time, `USD` for currency, `degrees_celsius` for temperature).
- **Timestamp**: A specific timestamp indicating when the datapoint was recorded or measured, or the start of the period it represents.

{% .callout type="note" %}
Consistency in naming conventions (e.g., using `snake_case` for keys like `page_load_time_ms`) and clear definitions for each datapoint are vital for a maintainable and understandable schema.
{% / .callout %}

## Datapoint Schema Fields

The following sections detail the schema of the `data_points` table as typically defined in a ClickHouse SQL schema for timeseries data. Each field is described along with its data type and purpose, providing a comprehensive structure for storing and querying datapoints.

### Timeseries and Entity Identification

These fields link the datapoint to a specific timeseries and the primary entity it describes.

| Name             | Required | Data Type | Description                                                                                                                                                              |
|------------------|----------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `series_gid`     |          | `UUID`    | The Global Unique Identifier (GID) of the metric or timeseries this datapoint belongs to. This typically links to an Entity that defines the metric itself.              |
| `entity_gid`     |          | `UUID`    | The GID of the specific entity that this datapoint is reporting on or is an attribute of (e.g., GID of a specific weather station). This links to an Entity.             |
| `entity_gid_url` |          | `String`  | The GID URL representation for the `entity_gid`, providing a resolvable link to the entity.                                                                                |

### Location Information

Provides geographical context for the datapoint.

| Name      | Required | Data Type | Description                                                                                                                                                                                                                            |
|-----------|----------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `geohash` |          | `String`  | The geolocation of the entity reported on at the time of the datapoint, represented as a geohash. This is useful for clustering, sorting, and spatial queries.<br/>{% .callout type="note" %}For non-stationary entities, the `geohash` will change over time.{% / .callout %} |

### Reporting Period/Intervals

Defines the temporal context of the datapoint.

| Name        | Required | Data Type | Description                                                                                                                                                                                                                                                           |
|-------------|----------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `period`    |          | `String`  | The resolution or frequency of the data, expressed in ISO 8601 duration format (e.g., `PT1H` for hourly, `P1D` for daily). This indicates the length of the interval for which the datapoint is valid or aggregated.                                                    |
| `timestamp` |          | `DateTime`| The calendar date and time (UTC) marking the beginning of the `period` for which the data is reported.<br/>{% .callout type="important" %}This is the actual timestamp of the measurement or observation (the start of the interval), not the ingestion time.{% / .callout %} |

### Ownership and Source

Tracks the provenance of the datapoint. All GIDs link to corresponding Entities.

| Name              | Required | Data Type | Description                                                                                                                               |
|-------------------|----------|-----------|-------------------------------------------------------------------------------------------------------------------------------------------|
| `owner_gid`       |          | `UUID`    | The GID of the entity that owns this datapoint (e.g., the organization responsible for the data).                                         |
| `source_gid`      |          | `UUID`    | The GID of the entity representing the original source of the data (e.g., a specific sensor, an external data provider).                   |
| `publisher_gid`   |          | `UUID`    | The GID of the entity that published this datapoint (e.g., the platform that made the data available).                                    |
| `publication_gid` |          | `UUID`    | The GID of the specific publication or dataset this datapoint belongs to. This can be useful for versioning or grouping data releases.     |

### Structured Metadata

These map fields allow for enriching datapoints with additional structured key-value details.

| Name             | Required | Data Type        | Description                                                                                                                                                           |
|------------------|----------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `gids`           |          | `Map(String, UUID)`   | Provides additional links to Named Entities, associating the datapoint with other relevant entities beyond `entity_gid`. Example: `{"related_campaign": "01234567-89ab-cdef-0123-456789abcdef"}`. |
| `location`       |          | `Map(String, String)` | Stores additional geographical details specific to this datapoint, supplementing `geohash`. Example: `{"venue_section": "A1"}`.                                    |
| `demography`     |          | `Map(String, String)` | Adds demographic information relevant to the datapoint. Example: `{"age_group": "25-34"}`.                                                                         |
| `classification` |          | `Map(String, String)` | Provides further classification details for the datapoint. Example: `{"data_sensitivity": "confidential"}`.                                                     |
| `topology`       |          | `Map(String, String)` | Describes network or system topology relevant to the datapoint. Example: `{"server_node": "node_101"}`.                                                             |
| `usage`          |          | `Map(String, String)` | Captures details about the usage context of the entity or system related to this datapoint. Example: `{"user_activity_level": "high"}`.                           |
| `device`         |          | `Map(String, String)` | Specifies details about the device from which the datapoint was generated or to which it pertains. Example: `{"device_model": "iPhone 15 Pro"}`.                     |
| `product`        |          | `Map(String, String)` | Contains additional product-related information relevant to the datapoint. Example: `{"product_sku": "SKU12345"}`.                                                |

### Flags and Tags

Offer flexible ways to add boolean markers or categorical labels.

| Name    | Required | Data Type        | Description                                                                                                                                                           |
|---------|----------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `flags` |          | `Map(String, Boolean)`| A map of named boolean flags for setting various binary characteristics. Example: `{"is_anomaly": true, "needs_review": false}`.                                      |
| `tags`  |          | `Array(String)`  | An array of string tags for applying multiple labels or keywords. Example: `["critical", "realtime_processed"]`.                                                        |

### Core Datapoint Information

These fields store the primary data values of the datapoint.

| Name         | Required | Data Type        | Description                                                                                                                                                                 |
|--------------|----------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `dimensions` |          | `Map(String, String)` | A map of key-value pairs representing dimensional attributes used for grouping/filtering. Example: `{"country": "US", "device_category": "mobile"}`.                     |
| `metrics`    |          | `Map(String, Float64)`| A map where keys are metric names (e.g., `sales_total`) and values are their `Float64` measurements. Example: `{"page_views": 1200.0, "cpu_utilization": 75.5}`.         |

### Measurement Details

Provide context about the `metrics`. Keys in these maps typically correspond to metric names in the `metrics` field.

| Name         | Required | Data Type        | Description                                                                                                                                                                                                                            |
|--------------|----------|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `mtype`      |          | `Map(String, String)` | Specifies the measurement type for each metric (e.g., counter, gauge). Example: `{"page_views": "counter"}`.                                                                                                                      |
| `uom`        |          | `Map(String, String)` | Defines the Unit of Measure for each metric.<br/>{% .callout type="info" %}All UOM values should link to a standardized UOM Entity.{% / .callout %} Example: `{"page_views": "Count", "cpu_utilization": "Percentage"}`.                 |
| `of_what`    |          | `Map(String, String)` | Describes *what* the metric is measuring, often linking to a standard definition. Key is metric name, value is the standard. Example: `{"population": "wd:Q11517"}`.                                                                  |
| `agg_method` |          | `Map(String, String)` | Specifies the default aggregation method for each metric (e.g., `sum`, `avg`). Key is metric name. Example: `{"page_views": "sum", "cpu_utilization": "avg"}`.                                                                     |

### Access, Partitioning, and System Fields

System-managed fields for data access, storage, and integrity.

| Name          | Required | Data Type | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|---------------|----------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `signature`   |          | `UUID`    | A UUID signature generated from `series_gid`, `dimensions`, `of_what`, and `metrics` keys.<br/>{% .callout type="important" %}Used internally by `ReplacingMergeTree` to manage updates/replacements of records.{% / .callout %}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `access_type` |          | `Enum`    | Defines the access control level. Default: `Exclusive` (1). Values: `Local` (0), `Exclusive` (1), `Group` (2), `SharedPercentiles` (3), `SharedObfuscated` (4), `Shared` (5), `Public` (6). The specific enum values like `'Local' = 0` are defined in the SQL schema.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `partition`   |          | `String`  | Storage partition key, often derived from `timestamp` (e.g., `YYYY-MM`). The SQL schema comment "The version of the event message" might imply use in data versioning or batch ID for partitioning.<br/>{% .callout type="info" %}Primarily for database performance and data lifecycle management.{% / .callout %}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `sign`        |          | `Int8`    | Internal field for ClickHouse's `ReplacingMergeTree` engine. Default is `1`.<br/>{% .callout type="info" %}The `sign` field (1 for active, -1 for deleted/updated versions) is essential for processing replacements and maintaining the latest record version during merges. Not typically user-set.{% / .callout %}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |

## Common Datapoint Examples

While the schema above defines the structure for storage, datapoints in practice often refer to individual metric values or descriptive attributes. Here are some illustrative examples of what a "datapoint" might conceptually represent:

- **Quantitative Datapoints** (representing measurements):
    - `order_value: 120.50` (with unit "USD")
    - `response_time_ms: 300` (with unit "milliseconds")
    - `items_in_cart: 3`
    - `temperature_celsius: 22.5`
    - `session_duration_seconds: 1800`

- **Descriptive Datapoints** (providing textual or qualitative information):
    - `button_color: "blue"`
    - `user_feedback_text: "The interface is very intuitive!"`
    - `product_name: "Organic Green Tea"`
    - `error_message: "Invalid input provided"`

- **Categorical Datapoints** (representing a selection from a predefined set of options):
    - `payment_method: "credit_card"`
    - `content_category: "news"`
    - `ui_theme: "dark_mode"`
    - `user_segment: "premium_tier"`

## How Datapoints Relate to Events and Entities

Datapoints are the atomic pieces of information that typically:

1.  Are collected as **part of a Semantic Event's payload**: They provide specific details about the occurrence itself. For example, an `item_viewed` event might include datapoints like `view_duration`, `scroll_depth_percentage`, and `item_price` as part of its properties.
2.  Describe the **attributes or state of an Entity at a point in time**: A `user` entity might have datapoints representing `total_purchase_count`, `last_login_date`, or `account_status`. These can be stored as individual records in the `data_points` table, linked to the user's `entity_gid`.

Essentially, datapoints are the fine-grained pieces of information that populate the attributes of events and describe entities, making them rich with detail for analysis.

## Benefits of Well-Defined Datapoints

```json
{
  "event": "Item Added to Cart",
  "entity_id": "user_789", // Related entity (User)
  "timestamp": "2023-10-27T10:30:00Z",
  // ... other event-level fields
  "properties": { // Datapoints often reside here, detailing the event
    "item_id": "product_xyz", // Identifier for related product entity
    "item_name": "Super Widget", // Descriptive datapoint
    "quantity": 2, // Quantitative datapoint
    "price_per_item": 25.99, // Quantitative datapoint
    "currency": "USD", // Categorical/Descriptive datapoint (context for price)
    "cart_total_items": 5, // Quantitative datapoint (state after event)
    "added_from_page": "/products/category/widgets" // Descriptive datapoint
  }
}
```

{% .callout type="info" %}
**Note:** The JSON above is a conceptual illustration of how datapoints might appear within an event's properties. The `data_points` table schema described earlier is designed for storing these granular facts in a structured timeseries database.
{% / .callout %}

## Generic Datapoint Structure Example

The following is an example of a single datapoint record as it might be stored in the `data_points` table, reflecting the schema fields described above:

```json
{
  "series_gid": "ts_01h4g3f2j1k0p0n8m7q5r4e3w2a1s0d1",
  "entity_gid": "ent_a1b2c3d4-e5f6-7890-1234-567890abcdef",
  "entity_gid_url": "nova://entities/ent_a1b2c3d4-e5f6-7890-1234-567890abcdef",
  "geohash": "u4pruydqqvj",
  "period": "PT1H",
  "timestamp": "2024-01-15T14:00:00Z",
  "owner_gid": "org_k9l8m7n6-p5o4-3210-fedc-ba0987654321",
  "source_gid": "src_z1y2x3w4-v5u6-7890-abcd-ef1234567890",
  "publisher_gid": "pub_s9r8q7p6-o5n4-m3l2-k1j0-ihgfedcba987",
  "publication_gid": "pubset_f1e2d3c4-b5a6-0987-7654-3210fedcba09",
  "dimensions": {
    "region": "emea",
    "product_category": "electronics"
  },
  "metrics": {
    "sales_total": 15750.75,
    "active_users": 12345.0
  },
  "mtype": {
    "sales_total": "gauge",
    "active_users": "gauge"
  },
  "uom": {
    "sales_total": "USD",
    "active_users": "count"
  },
  "of_what": {
    "sales_total": "concept:TotalRevenue",
    "active_users": "concept:ActiveUserCount"
  },
  "agg_method": {
    "sales_total": "sum",
    "active_users": "sum"
  },
  "gids": {
    "promotion_applied": "promo_smmr24_q12w"
  },
  "location": {
    "store_id": "STR0123",
    "country_iso": "DE"
  },
  "demography": {
    "age_bracket": "25-35"
  },
  "classification": {
    "customer_segment": "premium"
  },
  "topology": {
    "server_farm": "farm_eu_west_03"
  },
  "usage": {
    "data_plan": "unlimited"
  },
  "device": {
    "os_type": "android"
  },
  "product": {
    "product_id": "PRDXYZ789"
  },
  "flags": {
    "is_forecast_data": false,
    "is_aggregated": true
  },
  "tags": [
    "hourly_rollup",
    "emea_sales_data",
    "electronics_department"
  ],
  "signature": "sig_0a1b2c3d-4e5f-6a7b-8c9d-0e1f2a3b4c5d",
  "access_type": 5,
  "partition": "2024-01",
  "sign": 1
}
```
{% .callout type="note" %}
In this example, `access_type: 5` corresponds to 'Shared'. The `sign` and `partition` fields are typically managed by the system. This structure provides a rich, detailed view of a single datapoint, ready for complex analytics and timeseries analysis.
{% / .callout %}

Incorporating well-defined datapoints into your semantic event and entity structures offers significant benefits:

- **Precision in Data**: Datapoints capture the exact, low-level details, avoiding ambiguity.
- **Detailed Analysis**: They enable granular analysis, allowing you to drill down into specific aspects of events and entity behaviors.
- **Tracking Specific Metrics**: Key Performance Indicators (KPIs) and other metrics are often derived directly from specific datapoints.
- **Foundation for Rich Insights**: Aggregated and correlated datapoints form the basis for discovering trends, patterns, and actionable insights.
- **Enhanced Machine Learning**: Datapoints serve as critical features for training machine learning models for prediction, classification, or personalization.

By carefully defining and collecting relevant datapoints, organizations can ensure their data is not just voluminous but also rich in detail and value, powering more sophisticated analytics and data-driven actions.
