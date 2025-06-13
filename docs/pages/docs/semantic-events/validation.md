---
title: Event Validation
description: Validate semantic events against the Avro schema and event modeling rules
---

# Event Validation

The Context Suite includes a robust event validation system that ensures events conform to both the Avro schema definition and event modeling best practices. This validator is dynamically driven by the Avro schema, ensuring that validation rules evolve automatically as the schema is updated.

## Features

The validator provides comprehensive validation of event JSON objects:

1. **Required Field Validation**: Ensures presence of schema-defined required fields
2. **Type Validation**: Verifies field types match schema expectations
3. **Event Modeling Rules**: Validates compliance with best practices:
   - Property order recommendations
   - Entity references in `involves` array
   - Nested structure validation
4. **Unknown Field Detection**: Identifies fields not defined in the Avro schema, including in nested structures

## How It Works

The validator processes event validation in three steps:

1. **Avro Schema Parsing**: Extracts field definitions, types, and requirements directly from the Avro schema files (`.avsc`)
2. **Event Analysis**: Processes the event structure against schema requirements
3. **Validation Results**: Provides clear error/warning messages with specific details

Unlike previous validation approaches, this validator is dynamically driven by the Avro schema files, ensuring that validation rules automatically evolve as the schema is updated without requiring code changes.

## File Structure

The validator consists of three primary components:

- **avro_schema_parser.py**: Parses the Avro schemas to extract field definitions and requirements
- **event_validator.py**: Core validation logic that checks events against schema rules
- **run_validator.py**: CLI tool to validate JSON event files

These files are located in the `cxs-schema/` directory of the project.

## Usage

### Command Line Interface

The validator can be run directly from the command line:

```bash
# Validate a single event file
python cxs-schema/run_validator.py examples/events/product_updated.json

# Validate all events in a directory
python cxs-schema/run_validator.py -d examples/events/
```

### Python API

You can also use the validator programmatically in your Python code:

```python
from cxs_schema.event_validator import validate_event_file, validate_events

# Validate a single file
is_valid, errors, warnings = validate_event_file('path/to/event.json')

# Validate a list of events
events = [
    {"type": "semantic_events", "event": "product_updated", ...},
    {"type": "semantic_events", "event": "user_login", ...}
]
results = validate_events(events)
```

## Validation Rules

### Required Fields

The validator enforces the presence of all required fields from the SQL schema:
- `type`
- `event`
- `timestamp`
- `entity_gid`
- `event_gid`

```json
{
  "type": "semantic_events",
  "event": "product_updated",
  "timestamp": "2023-04-18T15:22:31.000Z",
  "entity_gid": "gid://Product/123",
  "event_gid": "gid://Event/456"
  // ... other fields
}
```

### Entity References

All fields ending with `_id` (except `message_id`) should be moved to the `involves` array with proper roles and entity types:

```json
{
  // ... other fields
  "involves": [
    {
      "entity_type": "User",
      "id": "123",
      "id_type": "database",
      "role": "Actor"
    },
    {
      "entity_type": "Product",
      "id": "456",
      "id_type": "database",
      "role": "Subject"
    }
  ]
}
```

### Property Ordering

For improved readability, the validator recommends that events follow a standard property order:

1. First properties: `type`, `event`, `timestamp`, `entity_gid`, `event_gid`
2. Last properties: `source`/`source_info`, `partition`, `integration`, `analysis`, `message_id`

These ordering rules generate warnings but do not cause validation failures.

### Nested Fields

Nested structures like `commerce.products`, `involves`, and `classification` are validated against their schema-defined fields:

```json
{
  // ... other fields
  "commerce": {
    "products": [
      {
        "id": "product123",
        "name": "Sample Product",
        "category": "Clothing",
        "url": "https://example.com/products/123"
      }
    ]
  }
}
```

## Edge Cases

The validator handles several edge cases:

1. **Map Fields**: Special handling for fields like `flags`, `properties` which can have arbitrary keys
2. **Content Fields**: Fields like `properties` and `content` can contain arbitrary structures
3. **Nullable Fields**: Handles nullable fields from the SQL schema appropriately
4. **Nested Structures**: Properly validates nested arrays and objects, including multi-level nesting

## Examples

### Valid Event Example

Below is a complete, validated example of a product_updated event that passes all validation checks against the Avro schema:

```json
{
  "type": "track",
  "event": "Product Updated",
  "timestamp": "2025-06-06T17:32:46-04:00",
  "entity_gid": "ede9ed75-dd83-519b-89c2-aa20966962cd",
  "event_gid": "ede9ed75-dd83-519b-89c2-aa20966962cd",
  "content": {
    "Description": "This is test snowboard",
    "Title": "The test snowboard"
  },
  "context": {
    "ip": "127.0.0.1",
    "locale": "en-US",
    "timezone": "America/Los_Angeles"
  },
  "flags": {
    "is_taxable": true
  },
  "involves": [
    {
      "entity_type": "Product",
      "id": "15100602155333",
      "id_type": "Shopify",
      "label": "The test snowboard",
      "role": "Source"
    },
    {
      "entity_type": "Category",
      "id": "sg-4-17-2-17",
      "id_type": "ShopifyCategory",
      "label": "Snowboards",
      "role": "Classification"
    }
  ],
  "classification": [
    {
      "type": "Category",
      "value": "Sporting Goods"
    },
    {
      "type": "Category",
      "value": "Outdoor Recreation"
    },
    {
      "type": "Category",
      "value": "Winter Sports & Activities"
    },
    {
      "type": "Category",
      "value": "Skiing & Snowboarding"
    },
    {
      "type": "Category",
      "value": "Snowboards"
    }
  ],
  "commerce": {
    "products": [
      {
        "product_id": "15100602155333",
        "product": "The test snowboard",
        "category": "Snowboards",
        "categories": [
          "Sporting Goods",
          "Outdoor Recreation", 
          "Winter Sports & Activities", 
          "Skiing & Snowboarding", 
          "Snowboards"
        ],
        "url": "gid://shopify/Product/15100602155333"
      }
    ]
  },
  "properties": {
    "admin_graphql_api_id": "gid://shopify/Product/15100602155333",
    "category_full_path": "Sporting Goods > Outdoor Recreation > Winter Sports & Activities > Skiing & Snowboarding > Snowboards"
  },
  "source_info": {
    "label": "Shopify",
    "type": "eCommerce"
  },
  "message_id": "shopify_product_15100602155333"
}
```

### Validation Output Examples

For a successful validation:

```
✓ product_updated_corrected.json: Validation passed!
```

For a validation with warnings:

```
⚠️ product_updated.json: Passed with warnings:
  1. Unknown field 'commerce.products[0].path' is not defined in SQL schema
```

For a validation failure:

```
❌ invalid_event.json: Validation failed:
  1. Missing required field 'event'
  2. Field 'timestamp' has invalid type: expected 'DateTime', got 'string'
  3. Unknown field 'user_data' is not defined in SQL schema
```

## Testing the Validator

You can test the validator with provided example events:

```bash
# Test with a valid event
python cxs-schema/run_validator.py examples/events/product_updated_corrected.json

# Test with an event that has warnings
python cxs-schema/run_validator.py examples/events/product_updated.json

# Test with multiple events
python cxs-schema/run_validator.py -d examples/events/
```

## Troubleshooting

If you encounter validation errors or warnings:

1. **Missing Required Fields**: Ensure all required fields from the SQL schema are present
2. **Type Errors**: Check that field types match schema expectations
3. **Unknown Fields**: Verify that all fields are defined in the SQL schema
4. **ID Fields**: Move all `*_id` fields into the `involves` array with appropriate roles
5. **Path Field**: The `path` field in `commerce.products` is not part of the SQL schema

For more detailed information about event modeling best practices, refer to the [Best Practices](/docs/semantic-events/best-practices) and [Events vs. Timestamps](/docs/semantic-events/events-vs-timestamps) documentation.
