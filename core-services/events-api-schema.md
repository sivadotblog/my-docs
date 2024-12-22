# Events API Schema (Draft)

## Core Entity: ApiEvent

### Data Model
```json
{
    "id": "uuid",
    "eventType": "string",
    "sourceSystem": "string",
    "eventTime": "datetime",
    "eventData": {
        "endpoint": "string",
        "method": "string",
        "requestBody": "object",
        "responseBody": "object",
        "statusCode": "integer"
    },
    "correlationId": "string",
    "userContext": {
        "userId": "string",
        "clientId": "string",
        "ipAddress": "string"
    },
    "status": "string",
    "created_at": "datetime",
    "updated_at": "datetime"
}
```

### Field Level Mapping
| Field Name | Data Type | Required | Description | Validation Rules | Default Value |
|------------|-----------|----------|-------------|------------------|---------------|
| id | UUID | Yes | Unique identifier for the event | Auto-generated | None |
| eventType | String | Yes | Type of API event (e.g., REQUEST, RESPONSE, ERROR) | max_length=50, enum values | None |
| sourceSystem | String | Yes | System/service that generated the event | max_length=100 | None |
| eventTime | DateTime | Yes | Time when the event occurred | ISO 8601 format | None |
| eventData.endpoint | String | Yes | API endpoint path | max_length=500 | None |
| eventData.method | String | Yes | HTTP method | enum: GET, POST, PUT, DELETE, etc. | None |
| eventData.requestBody | Object | No | Request payload | max_size=1MB | null |
| eventData.responseBody | Object | No | Response payload | max_size=1MB | null |
| eventData.statusCode | Integer | Yes | HTTP status code | range: 100-599 | None |
| correlationId | String | Yes | ID to correlate related events | UUID format | None |
| userContext.userId | String | No | ID of the user making the request | max_length=100 | null |
| userContext.clientId | String | No | ID of the client application | max_length=100 | null |
| userContext.ipAddress | String | No | IP address of the client | valid IP format | null |
| status | String | Yes | Overall event status | enum: SUCCESS, FAILURE | None |
| created_at | DateTime | Yes | Record creation timestamp | Auto-generated | Current timestamp |
| updated_at | DateTime | Yes | Record last update timestamp | Auto-generated | Current timestamp |

### Database Schema
```sql
CREATE TABLE api_events (
    id UUID PRIMARY KEY,
    event_type VARCHAR(50) NOT NULL,
    source_system VARCHAR(100) NOT NULL,
    event_time TIMESTAMP WITH TIME ZONE NOT NULL,
    event_data JSONB NOT NULL,
    correlation_id UUID NOT NULL,
    user_context JSONB,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT valid_event_type CHECK (event_type IN ('REQUEST', 'RESPONSE', 'ERROR')),
    CONSTRAINT valid_status CHECK (status IN ('SUCCESS', 'FAILURE'))
);

CREATE INDEX idx_api_events_correlation_id ON api_events(correlation_id);
CREATE INDEX idx_api_events_event_time ON api_events(event_time);
CREATE INDEX idx_api_events_source_system ON api_events(source_system);
```

### Validation Rules (Pydantic Model)
```python
from pydantic import BaseModel, Field, IPvAnyAddress
from typing import Optional, Dict, Any
from datetime import datetime
from uuid import UUID
from enum import Enum

class EventType(str, Enum):
    REQUEST = "REQUEST"
    RESPONSE = "RESPONSE"
    ERROR = "ERROR"

class Status(str, Enum):
    SUCCESS = "SUCCESS"
    FAILURE = "FAILURE"

class HttpMethod(str, Enum):
    GET = "GET"
    POST = "POST"
    PUT = "PUT"
    DELETE = "DELETE"
    PATCH = "PATCH"
    HEAD = "HEAD"
    OPTIONS = "OPTIONS"

class EventData(BaseModel):
    endpoint: str = Field(..., max_length=500)
    method: HttpMethod
    requestBody: Optional[Dict[str, Any]] = None
    responseBody: Optional[Dict[str, Any]] = None
    statusCode: int = Field(..., ge=100, le=599)

class UserContext(BaseModel):
    userId: Optional[str] = Field(None, max_length=100)
    clientId: Optional[str] = Field(None, max_length=100)
    ipAddress: Optional[IPvAnyAddress] = None

class ApiEvent(BaseModel):
    id: UUID
    eventType: EventType
    sourceSystem: str = Field(..., max_length=100)
    eventTime: datetime
    eventData: EventData
    correlationId: UUID
    userContext: Optional[UserContext] = None
    status: Status
    created_at: datetime
    updated_at: datetime
```

### Notes
1. The schema follows CDM principles by:
   - Using standardized data types
   - Including comprehensive audit fields
   - Supporting extensible JSON fields for event data and user context
   - Following consistent naming conventions

2. Performance considerations:
   - Indexes on frequently queried fields
   - JSONB type for flexible schema evolution
   - Partitioning by event_time could be added for large-scale deployments

3. Security considerations:
   - Sensitive data in event payloads should be masked/encrypted
   - IP addresses should be handled according to privacy regulations
   - User context helps with audit trails

Please review this schema. Key points to consider:
1. Is the granularity of event data sufficient?
2. Are there additional fields needed for your specific use case?
3. Should we add any additional indexes or constraints?
4. Are the validation rules appropriate?
