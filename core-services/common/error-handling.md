# Error Handling Standards

## 1. Error Response Format

### 1.1 Standard Error Response
```python
from pydantic import BaseModel
from typing import Optional, Dict, Any
from datetime import datetime
import uuid

class ErrorDetail(BaseModel):
    reason: str
    suggestion: Optional[str] = None

class ErrorResponse(BaseModel):
    code: str
    message: str
    details: Optional[ErrorDetail] = None
    correlation_id: str = str(uuid.uuid4())
    timestamp: str = datetime.utcnow().isoformat()

    class Config:
        json_schema_extra = {
            "example": {
                "code": "ERR_1000",
                "message": "Authentication required",
                "details": {
                    "reason": "No valid token provided",
                    "suggestion": "Include a valid Bearer token in Authorization header"
                },
                "correlation_id": "123e4567-e89b-12d3-a456-426614174000",
                "timestamp": "2023-01-01T00:00:00Z"
            }
        }
```

## 2. Common Error Codes

### 2.1 Authentication Errors (1xxx)
| Code    | Message                   | HTTP Status | Description                                    |
|---------|---------------------------|-------------|------------------------------------------------|
| ERR_1000| Authentication Required   | 401        | No Azure AD token provided                     |
| ERR_1001| Invalid Token            | 401        | Azure AD token is invalid                      |
| ERR_1002| Insufficient Scope       | 403        | Token lacks required scope                     |

### 2.2 Validation Errors (2xxx)
| Code    | Message                   | HTTP Status | Description                                    |
|---------|---------------------------|-------------|------------------------------------------------|
| ERR_2000| Invalid Input            | 400        | Request validation failed                      |
| ERR_2001| Missing Required Field   | 400        | Required field not provided                    |
| ERR_2002| Invalid Format           | 400        | Field format is incorrect                      |

### 2.3 Resource Errors (3xxx)
| Code    | Message                   | HTTP Status | Description                                    |
|---------|---------------------------|-------------|------------------------------------------------|
| ERR_3000| Resource Not Found       | 404        | Requested resource doesn't exist               |
| ERR_3001| Resource Exists          | 409        | Resource already exists                        |
| ERR_3002| Resource Locked          | 423        | Resource is currently locked                   |

### 2.4 System Errors (4xxx)
| Code    | Message                   | HTTP Status | Description                                    |
|---------|---------------------------|-------------|------------------------------------------------|
| ERR_4000| Internal Server Error    | 500        | Unexpected server error                        |
| ERR_4001| Service Unavailable      | 503        | Service temporarily unavailable                |
| ERR_4002| Dependency Error         | 502        | External service call failed                   |

## 3. Error Handling Implementation

### 3.1 Custom Exception Classes
```python
from fastapi import HTTPException
from typing import Optional

class APIError(HTTPException):
    def __init__(
        self,
        code: str,
        message: str,
        status_code: int,
        details: Optional[ErrorDetail] = None
    ):
        self.error_response = ErrorResponse(
            code=code,
            message=message,
            details=details
        )
        super().__init__(
            status_code=status_code,
            detail=self.error_response.dict()
        )

class AuthenticationError(APIError):
    def __init__(
        self, 
        message: str = "Authentication required",
        code: str = "ERR_1000",
        details: Optional[ErrorDetail] = None
    ):
        super().__init__(code, message, 401, details)

class ValidationError(APIError):
    def __init__(
        self,
        message: str = "Validation error",
        code: str = "ERR_2000",
        details: Optional[ErrorDetail] = None
    ):
        super().__init__(code, message, 400, details)

class ResourceNotFoundError(APIError):
    def __init__(
        self,
        resource: str,
        code: str = "ERR_3000",
        details: Optional[ErrorDetail] = None
    ):
        message = f"{resource} not found"
        super().__init__(code, message, 404, details)
```

### 3.2 FastAPI Exception Handlers
```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(APIError)
async def api_error_handler(request: Request, exc: APIError) -> JSONResponse:
    return JSONResponse(
        status_code=exc.status_code,
        content=exc.detail
    )

@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception) -> JSONResponse:
    error_response = ErrorResponse(
        code="ERR_4000",
        message="An unexpected error occurred",
        details=ErrorDetail(
            reason=str(exc)
        ) if app.debug else None
    )
    return JSONResponse(
        status_code=500,
        content=error_response.dict()
    )
```

### 3.3 Usage Examples
```python
from fastapi import FastAPI, Depends, Path
from typing import Dict

app = FastAPI()

@app.get("/api/users/{user_id}")
async def get_user(user_id: str = Path(..., title="The ID of the user to get")):
    try:
        user = await user_service.get_user(user_id)
        if not user:
            raise ResourceNotFoundError(
                resource="User",
                details=ErrorDetail(
                    reason=f"User with ID {user_id} does not exist",
                    suggestion="Please verify the user ID and try again"
                )
            )
        return user
    except ValidationError:
        raise
    except Exception as e:
        # Log the error with correlation ID
        logger.error(
            "Error fetching user",
            extra={
                "user_id": user_id,
                "error": str(e),
                "correlation_id": request.state.correlation_id
            }
        )
        raise
```

## 4. Error Handling Guidelines

### 4.1 Best Practices
1. Always use standard error response format
2. Include correlation IDs for tracking
3. Log errors with appropriate context
4. Avoid exposing internal details in production
5. Use appropriate HTTP status codes
6. Include helpful error messages and suggestions

### 4.2 Error Logging
```python
import logging
from contextvars import ContextVar
from typing import Optional

# Context variable for correlation ID
correlation_id: ContextVar[str] = ContextVar('correlation_id')

class APILogger:
    def __init__(self, name: str):
        self.logger = logging.getLogger(name)
        
    def error(
        self,
        message: str,
        error: Optional[Exception] = None,
        **kwargs
    ):
        try:
            correlation = correlation_id.get()
        except LookupError:
            correlation = str(uuid.uuid4())
            
        self.logger.error(
            message,
            extra={
                "correlation_id": correlation,
                "error_type": type(error).__name__ if error else None,
                "error_message": str(error) if error else None,
                **kwargs
            },
            exc_info=error is not None
        )

# Usage
logger = APILogger(__name__)
```

### 4.3 Client Retry Guidelines
| Error Code Range | Retry Recommended | Retry Strategy |
|-----------------|-------------------|----------------|
| 1xxx            | No               | N/A            |
| 2xxx            | No               | N/A            |
| 3xxx            | Conditional      | Based on error |
| 4xxx            | Yes              | Exponential backoff |

### 4.4 Dependencies
```python
# requirements.txt
fastapi==0.104.1
pydantic==2.4.2
python-jose[cryptography]==3.3.0
python-multipart==0.0.6
python-dotenv==0.21.0
uvicorn==0.22.0
