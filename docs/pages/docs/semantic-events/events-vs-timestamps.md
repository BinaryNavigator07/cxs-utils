---
title: Events vs Timestamps
---

# Events vs Timestamps in Context Suite

## Events as First-Class Citizens

In Context Suite, events should be treated as "first-class citizens" rather than being reduced to simple timestamp fields. This represents a fundamental shift in how we think about data:

### Traditional Approach (State-Based)

In traditional systems, changes are often represented as timestamp fields:

```json
"created_at": "2025-06-06T17:32:44-04:00"
"updated_at": "2025-06-06T17:32:46-04:00"
"published_at": "2025-06-06T17:31:23-04:00"
```

These timestamps reduce rich events to mere state changes, losing valuable context and information.

### Context Suite Approach (Event-Based)

Instead of timestamps, Context Suite promotes explicit events:

- "Product Created"
- "Product Updated"
- "Product Published"

Each event carries complete context, including who performed the action, what changed, and why.

## The Problem with Timestamp Reduction

When you see timestamp fields like `created_at`, `updated_at`, or `published_at`, you should recognize these as "events reduced to state changes." This reduction:

1. Loses valuable context (who made the change, why, what specifically changed)
2. Makes analysis more difficult
3. Prevents rich linking between related events
4. Limits the ability to build comprehensive event timelines

## Roles in the Involves Structure

The `role` field in the `involves` structure is particularly important and often misunderstood:

### Role vs. Entity Type

- **Entity Type**: Describes what the entity *is* (e.g., "Person", "Product", "ProductVariant")
- **Role**: Describes how the entity is *involved* in this specific event

The role is rarely a repeat of the entity type. Instead, it explains the entity's involvement in the event.

### Example: Car Crash Event

In a car crash event, multiple entities might be of type "Person" but have different roles:

- Role: "Driver" (Entity Type: "Person")
- Role: "Pedestrian" (Entity Type: "Person")
- Role: "Witness" (Entity Type: "Person")

### Special Roles

- **Source**: The entity that triggered the event
- **Parent**: A hierarchical relationship (e.g., Product is the Parent of ProductVariant)

### Example: Product Variant Event

When a Product Variant fires an event:

- The variant is the "Source" (it triggered the event)
- The Product is the "Parent" (hierarchical relationship)

## IDs and References

When you see `_id` fields (like `inventory_item_id`), these should be thought of as references to other entities that should be included in the `involves` structure.

Instead of:
```json
"inventory_item_id": "54319739470149"
```

Consider using:
```json
"involves": [
  {
    "label": "Inventory Item",
    "role": "InventoryItem", 
    "entity_type": "InventoryItem",
    "id": "54319739470149",
    "id_type": "Shopify"
  }
]
```

## Benefits of This Approach

1. **Complete Context**: Each event carries full information about what happened
2. **Rich Relationships**: Events explicitly link to all involved entities
3. **Better Analysis**: AI/ML systems can work with fully enriched data
4. **Temporal Understanding**: Events can be properly sequenced and understood in context
5. **Consistent Structure**: All events follow the same pattern, making analysis easier

## Practical Application

When designing systems with Context Suite:

1. Convert timestamp fields into explicit events
2. Use the `involves` structure to link all relevant entities
3. Assign meaningful roles that describe involvement, not just entity types
4. Make sure events contain all necessary context
5. Think of events as the primary data structure, not just metadata about state changes

This event-centric approach provides a much richer foundation for analysis, especially for AI/ML systems that benefit from fully enriched and linked events.
