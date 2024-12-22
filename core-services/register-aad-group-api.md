# Register AAD Group API Documentation

[Previous content remains the same until the Request Body example]

##### Request Body
```json
{
    "groupName": "az_adb_data_scientists",
    "owner": {
        "id": "12345678-1234-5678-1234-567812345678",
        "email": "john.doe@example.com"
    },
    "scim_app": "unity_catalog"
}
```

##### Response (202 Accepted)
```json
{
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "groupName": "az_adb_data_scientists",
    "owner": {
        "id": "12345678-1234-5678-1234-567812345678",
        "email": "john.doe@example.com"
    },
    "scim_app": "unity_catalog",
    "aadStatus": {
        "status": "PROCESSING"
    },
    "ownerStatus": {
        "status": "PROCESSING"
    },
    "scimStatus": {
        "status": "PROCESSING"
    },
    "created_at": "2024-01-20T10:30:00Z",
    "updated_at": "2024-01-20T10:30:00Z"
}
```

##### Error Responses
| Status Code | Error Code | Description |
|-------------|------------|-------------|
| 400 | ERR_2000 | Invalid input |
| 400 | ERR_2001 | Invalid group name prefix for SCIM app |
| 401 | ERR_1000 | Authentication required |
| 403 | ERR_1002 | Insufficient scope |
| 409 | ERR_4000 | Group name already exists |

[Previous content remains the same until Business Rules section]

### 6.2 Business Rules
- Group names must be unique across the system
- Group names must follow Azure naming conventions
- Group name must start with one of the allowed prefixes for the specified SCIM app
  - Example: For SCIM app "unity_catalog", group name must start with either "az_adb_" or "az_databricks_"
  - This ensures proper routing and access management in target systems
  - The allowed prefixes are retrieved from the SCIM Configuration API
  - Invalid prefix combinations will be rejected (ERR_2001)
- Owner must be a valid AAD user
- SCIM app must exist in SCIM Configuration API
- All status transitions must be tracked
- Error messages must be preserved for troubleshooting

### 6.3 Example Prefix Validation
```python
async def validate_group_name_prefix(group_name: str, scim_app: str) -> bool:
    # Get SCIM app configuration
    scim_config = await get_scim_config(scim_app)
    if not scim_config:
        raise ValueError("SCIM app not found")
    
    # Check if group name starts with any allowed prefix
    return any(
        group_name.startswith(prefix)
        for prefix in scim_config.allowed_prefixes
    )

# Usage Example:
# unity_catalog prefixes: ["az_adb_", "az_databricks_"]
# Valid: "az_adb_data_scientists" ✓
# Valid: "az_databricks_engineers" ✓
# Invalid: "data_scientists" ✗
# Invalid: "dbx_analysts" ✗
```

[Rest of the content remains the same]
