# Microservice API Requirements Documentation

## 1. Service Overview

### 1.1 Purpose
[Describe the main purpose and business value of this microservice]

### 1.2 Scope
#### In Scope
- [List functionalities included in this service]
- [Define clear boundaries of responsibility]
- [Specify key features]

#### Out of Scope
- [List what this service explicitly does not handle]
- [Clarify boundaries with other services]

### 1.3 Service Dependencies
- **Upstream Services:**
  - [List services that call this service]
- **Downstream Dependencies:**
  - [List services this service calls]
  - [Include external APIs/systems]

## 2. Data Model

### 2.1 Core Entities
#### [Entity Name]
```json
{
    "id": "uuid",
    "field1": "string",
    "field2": "number",
    "created_at": "datetime",
    "updated_at": "datetime"
}
```

### 2.2 Field Level Mapping
| Field Name | Data Type | Required | Description | Validation Rules | Default Value |
|------------|-----------|----------|-------------|------------------|---------------|
| id         | UUID      | Yes      | Unique identifier | Auto-generated | None |
| field1     | String    | Yes      | Description | max_length=100 | None |
| field2     | Number    | No       | Description | min=0, max=100 | 0 |

### 2.3 Database Schema
```sql
CREATE TABLE entity_name (
    id UUID PRIMARY KEY,
    field1 VARCHAR(100) NOT NULL,
    field2 INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

## 3. API Endpoints

### 3.1 Base URL
```
https://api.example.com/v1/[service-name]
```

### 3.2 Endpoint Definitions

#### Create [Resource]
- **Method:** POST
- **Path:** `/resource`
- **Required Scope:** `resource.write`

##### Request Body
```json
{
    "field1": "string",
    "field2": "number"
}
```

##### Response (201 Created)
```json
{
    "id": "uuid",
    "field1": "string",
    "field2": "number",
    "created_at": "datetime",
    "updated_at": "datetime"
}
```

##### Error Responses
| Status Code | Error Code | Description |
|-------------|------------|-------------|
| 400 | ERR_2000 | Invalid input |
| 401 | ERR_1000 | Authentication required |
| 403 | ERR_1002 | Insufficient scope |

#### Get [Resource]
- **Method:** GET
- **Path:** `/resource/{id}`
- **Required Scope:** `resource.read`

##### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | UUID | Yes | Resource identifier |

##### Response (200 OK)
```json
{
    "id": "uuid",
    "field1": "string",
    "field2": "number",
    "created_at": "datetime",
    "updated_at": "datetime"
}
```

##### Error Responses
| Status Code | Error Code | Description |
|-------------|------------|-------------|
| 401 | ERR_1000 | Authentication required |
| 403 | ERR_1002 | Insufficient scope |
| 404 | ERR_3000 | Resource not found |

## 4. Authentication & Authorization

### 4.1 Authentication
Refer to [common/auth-requirements.md](common/auth-requirements.md) for detailed authentication implementation.

### 4.2 Required Scopes
| Endpoint | HTTP Method | Required Scope | Description |
|----------|-------------|----------------|-------------|
| /resource | POST | resource.write | Create new resource |
| /resource/{id} | GET | resource.read | Read resource |
| /resource/{id} | PUT | resource.write | Update resource |
| /resource/{id} | DELETE | resource.write | Delete resource |

## 5. Error Handling
Refer to [common/error-handling.md](common/error-handling.md) for detailed error handling implementation.

## 6. Data Validation Rules

### 6.1 Input Validation
```python
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime
from uuid import UUID

class ResourceCreate(BaseModel):
    field1: str = Field(..., min_length=1, max_length=100)
    field2: Optional[int] = Field(None, ge=0, le=100)

class Resource(ResourceCreate):
    id: UUID
    created_at: datetime
    updated_at: datetime
```

### 6.2 Business Rules
- [List any business-specific validation rules]
- [Define constraints beyond basic data validation]
- [Specify any cross-field validations]

## 7. Performance Requirements

### 7.1 SLA Targets
- Response Time: [e.g., 95th percentile < 500ms]
- Throughput: [e.g., 100 requests/second]
- Availability: [e.g., 99.9%]

### 7.2 Resource Limits
- Max Request Size: [e.g., 1MB]
- Rate Limits: [e.g., 1000 requests per minute per client]
- Concurrent Requests: [e.g., 100 per client]

## 8. Security Requirements

### 8.1 Data Classification
- [Specify data sensitivity level]
- [List any PII or sensitive fields]
- [Define data retention requirements]

### 8.2 Security Controls
- [List required security measures]
- [Specify encryption requirements]
- [Define access control policies]

## 9. Development Guidelines

### 9.1 Code Organization
```
service_name/
├── api/
│   ├── routes/
│   ├── models/
│   └── dependencies/
├── core/
│   ├── config.py
│   └── security.py
├── db/
│   ├── models.py
│   └── repositories/
└── services/
    └── business_logic/
```

### 9.2 Development Setup
```bash
# Environment setup
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run migrations
alembic upgrade head

# Start service
uvicorn main:app --reload
```

## 10. Testing Requirements

### 10.1 Test Categories
1. Unit Tests
   - Business logic
   - Data validation
   - Error handling

2. Integration Tests
   - API endpoints
   - Database operations
   - External service interactions

3. Performance Tests
   - Load testing
   - Stress testing
   - Endurance testing

### 10.2 Test Data
- [Define test data requirements]
- [Specify test environment needs]
- [List required mock services]

## Appendix

### A. Changelog
| Version | Date | Changes | Author |
|---------|------|---------|---------|
| 1.0.0   | [Date] | Initial version | [Author] |

### B. References
- API Design Guidelines
- Service Architecture Documentation
- Related Services Documentation
