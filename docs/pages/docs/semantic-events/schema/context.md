---
title: Event Context Properties
---

# Event Context Properties

Semantic events are enriched with various contextual details that provide a fuller understanding of the circumstances surrounding the event. These properties range from general environmental information (like IP address and locale) to more specific, dynamic awareness of entities involved or related to the event.

This document details two main types of context properties:
1.  **General Event Context (`context.*` fields):** Standard contextual attributes, often prefixed with `context.`.
2.  **Contextual Awareness (`contextual_awareness` structure):** A nested structure designed to hold more dynamic or nuanced details about entities relevant to the event.

While some very basic context fields (like IP address or locale) might be summarized in high-level event views (and may have been historically mentioned in `core.md`), this document provides the comprehensive definition for these `context.*` properties and the `contextual_awareness` structure.

## General Event Context (`context.*`)

These fields provide common contextual information about the environment in which the event was generated or processed.

| Name               | Required | Data Type      | Description                                                                                                                                                              |
|--------------------|----------|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `context.active`   |          | `Boolean`      | Indicates if the event context (e.g., a user session or browser tab) was active when the event occurred. This field is optional.                                           |
| `context.ip`       |          | `String`       | The IPv4 address from which the event originated. This field is optional.                                                                                                  |
| `context.ipv6`     |          | `String`       | The IPv6 address from which the event originated. This field is optional.                                                                                                  |
| `context.locale`   |          | `String`       | The locale string for the event context (e.g., "en-US", "de-DE"). Originally `LowCardinality(String)`. This field is optional.                                              |
| `context.group_id` |          | `String`       | An identifier for a group associated with the context (e.g., an organization or project ID). Originally `LowCardinality(String)`. This field is optional.                 |
| `context.timezone` |          | `String`       | The timezone of the event context (e.g., "America/New_York", "Europe/Berlin"). Originally `LowCardinality(String)`. This field is optional.                                 |
| `context.location` |          | `Array(Double)`| Simplified geographic coordinates (often latitude, longitude) associated with the event's immediate context. This is distinct from the main `location` object. This field is optional. |
| `context.extras`   |          | `String`       | A JSON string or stringified object containing any other custom contextual information not covered by standard fields. This field is optional.                               |

**Use Cases for General Context:**
*   `context.ip` and `context.location` can be used for approximate geolocation of events.
*   `context.locale` and `context.timezone` help in localizing timestamps, currency, and content.
*   `context.extras` offers flexibility to include any other relevant, non-standard contextual data.

## Contextual Awareness (`contextual_awareness` structure)

The `contextual_awareness` structure provides a flexible way to attach more specific, often dynamic, pieces of information about entities that are relevant to understanding or processing an event. This is particularly useful for capturing transient states or calculated attributes of entities at the time of the event.

This field is typically a list (array) of `contextual_awareness` objects. Each object has the following properties:

| Name          | Required | Data Type | Description                                                                                                                                                              |
|---------------|----------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `type`        |          | `String`  | The type or category of contextual information being provided (e.g., "UserSegment", "DeviceCapability", "FeatureFlag"). Originally `LowCardinality(String)`.                 |
| `entity_type` |          | `String`  | The type of entity this contextual piece of information pertains to (e.g., "user", "device", "application"). Originally `LowCardinality(String)`.                           |
| `entity_gid`  |          | `UUID`    | The Global ID (GID) of the specific entity instance this context is about. This field is optional.                                                                         |
| `entity_wid`  |          | `String`  | Workspace ID or a similar namespaced identifier for the entity, if applicable. This field is optional.                                                                     |
| `context`     |          | `String`  | The actual contextual value or detail (e.g., "HighValueSegment", "has_nfc:true", "new_checkout_flow:enabled"). This field is optional.                                      |

**Use Cases for `contextual_awareness`:**

*   **Dynamic User Segmentation:** An event might include a `contextual_awareness` object like:
    *   `type: "UserSegment"`, `entity_type: "user"`, `entity_gid: "user-uuid-123"`, `context: "FrequentShopper_Last30Days"`
*   **Feature Flag Status:** Indicating whether a feature was enabled for a user when an event occurred:
    *   `type: "FeatureFlag"`, `entity_type: "user"`, `entity_gid: "user-uuid-123"`, `context: "NewDashboardFeature:true"`
*   **Device Capabilities:** Noting a specific capability of the user's device at the time of a mobile event:
    *   `type: "DeviceCapability"`, `entity_type: "device"`, `entity_gid: "device-uuid-456"`, `context: "biometric_auth_available"`
*   **Real-time System State:** Capturing a relevant state of a related system or entity:
    *   `type: "SystemHealth"`, `entity_type: "service"`, `entity_gid: "payment-gateway-gid"`, `context: "DegradedPerformance"`

The `contextual_awareness` structure allows for rich, dynamic annotations that can be critical for conditional logic, targeted analysis, and understanding the precise circumstances under which an event took place, especially in complex, interconnected systems.
