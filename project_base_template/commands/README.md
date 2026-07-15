<!--
Copyright (c) 2026 8848 Digital LLP. All rights reserved.
Proprietary and confidential. Unauthorized copying, distribution, or use
of this file, via any medium, is strictly prohibited without prior
written permission from 8848 Digital LLP.
-->

# Custom Export Fixtures Command

## Overview

- **`8848-export-fixtures`**: Custom command for exporting custom fixtures with enhanced data cleaning

### What it does
- Exports fixtures defined in the `custom_fixtures` hook
- Uses `frappe.get_hooks("custom_fixtures", app_name=app)` to get fixture definitions
- Exports data to `[app]/fixtures/` directory
- Removes fields with null, empty string, or zero values
- Automatically cleans up unnecessary data to reduce fixture size

## Example Usage

### Export all custom fixtures
```bash
bench --site <site> 8848-export-fixtures
```

### Export custom fixtures for specific app
```bash
bench --site <site> 8848-export-fixtures --app <app_name>
```


## Configuration Example

```python
# hooks.py
custom_fixtures = [
    # Export Custom Fields for specific module
    {"dt": "Custom Field", "filters": {"module": "<Module Name>"}},
]

commands = ["<app_name>.commands.export_fixtures.export_fixtures"]
```

## Key Differences Summary

| Feature | `export-fixtures` | `8848-export-fixtures` |
| ---------------------- | ----------------- | ---------------------- |
| **Hook Source** | `fixtures` | `custom_fixtures` |
| **Data Cleaning** | None | Automatic cleanup of empty/null values |
| **Fixture Size** | Larger (includes all data) | Smaller (optimized data) |


## Notes

- The custom command is designed to work alongside Frappe's built-in functionality
