# Events API Schema (Common Data Model)

This document defines the schema for the Events API using the Microsoft Common Data Model (CDM) standard. This API collects all API events and supports POST requests for creating new events and GET requests for retrieving events by ID and creation/update timestamp.

## CDM Entities

### Event

| Field Name          | Data Type   | Description                                            |
|---------------------|-------------|--------------------------------------------------------|
| EventId             | string (guid) | Unique identifier for the event.                       |
| EventType           | string        | Type of the event (e.g., "UserLogin", "DataChange"). |
| EventSource         | string        | Source of the event (e.g., "AuthService", "DataAPI"). |
| EventData           | object        | Data associated with the event (JSON payload).         |
| CreatedDateTime     | datetime      | Date and time when the event was created (UTC).      |
| ModifiedDateTime    | datetime      | Date and time when the event was last modified (UTC). |

## API Operations

### Create Event (POST)

Request Body:

```json
{
  "EventType": "string",
  "EventSource": "string",
  "EventData": {}
}
```

Response (201 Created):

```json
{
  "EventId": "guid",
  "EventType": "string",
  "EventSource": "string",
  "EventData": {},
  "CreatedDateTime": "datetime",
  "ModifiedDateTime": "datetime"
}
```

### Get Event by ID (GET)

Path Parameters:

| Parameter | Data Type   | Description                           |
|-----------|-------------|---------------------------------------|
| eventId   | string (guid) | Unique identifier of the event.       |

Response (200 OK):

```json
{
  "EventId": "guid",
  "EventType": "string",
  "EventSource": "string",
  "EventData": {},
  "CreatedDateTime": "datetime",
  "ModifiedDateTime": "datetime"
}
```

### Get Events by Timestamp (GET)

Query Parameters:

| Parameter     | Data Type | Description                                         |
|---------------|-----------|-----------------------------------------------------|
| createdAfter  | datetime  | Return events created after this timestamp (UTC).   |
| createdBefore | datetime  | Return events created before this timestamp (UTC). |
| modifiedAfter | datetime  | Return events modified after this timestamp (UTC). |
| modifiedBefore| datetime  | Return events modified before this timestamp (UTC). |

Response (200 OK):

```json
[
  {
    "EventId": "guid",
    "EventType": "string",
    "EventSource": "string",
    "EventData": {},
    "CreatedDateTime": "datetime",
    "ModifiedDateTime": "datetime"
  },
  {
    "EventId": "guid",
    "EventType": "string",
    "EventSource": "string",
    "EventData": {},
    "CreatedDateTime": "datetime",
    "ModifiedDateTime": "datetime"
  }
]
