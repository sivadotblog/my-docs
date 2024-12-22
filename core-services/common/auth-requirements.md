# Authentication & Authorization Requirements

## 1. Azure AD Integration

### 1.1 Authentication Method
- Azure Active Directory (Azure AD)
- OAuth 2.0 with JWT tokens handled by Microsoft
- Implementation using MSAL Python library with FastAPI

### 1.2 Integration Requirements
1. **Azure AD Configuration**
   - Register application in Azure AD
   - Configure allowed redirect URIs
   - Define API permissions

2. **Required Azure AD Settings**
   - Application (client) ID
   - Directory (tenant) ID
   - Client secret (for confidential clients)

### 1.3 MSAL Python Setup
```python
from fastapi import FastAPI, Depends, HTTPException, Security
from fastapi.security import OAuth2AuthorizationCodeBearer
from typing import Optional, List
import msal
import os

# MSAL configuration
CLIENT_ID = os.getenv("AZURE_CLIENT_ID")
CLIENT_SECRET = os.getenv("AZURE_CLIENT_SECRET")
TENANT_ID = os.getenv("AZURE_TENANT_ID")
AUTHORITY = f"https://login.microsoftonline.com/{TENANT_ID}"

# Initialize MSAL confidential client
msal_app = msal.ConfidentialClientApplication(
    client_id=CLIENT_ID,
    client_credential=CLIENT_SECRET,
    authority=AUTHORITY
)

# Initialize FastAPI
app = FastAPI()

# OAuth2 scheme for Swagger UI
oauth2_scheme = OAuth2AuthorizationCodeBearer(
    authorizationUrl=f"{AUTHORITY}/oauth2/v2.0/authorize",
    tokenUrl=f"{AUTHORITY}/oauth2/v2.0/token"
)
```

### 1.4 Token Validation
```python
from jose import jwt
import requests
from fastapi import Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def validate_token(
    credentials: HTTPAuthorizationCredentials = Security(security)
) -> dict:
    token = credentials.credentials
    try:
        # Get signing keys from Microsoft
        jwks_uri = f"{AUTHORITY}/discovery/v2.0/keys"
        jwks = requests.get(jwks_uri).json()
        
        # Decode and validate the token
        decoded = jwt.decode(
            token,
            jwks,
            algorithms=["RS256"],
            audience=CLIENT_ID,
            issuer=f"{AUTHORITY}/v2.0"
        )
        return decoded
    except Exception as e:
        raise HTTPException(
            status_code=401,
            detail=f"Invalid authentication token: {str(e)}"
        )
```

## 2. Authorization

### 2.1 Scope Definition
```python
from typing import List
from pydantic import BaseModel

class ScopeDefinition(BaseModel):
    name: str
    description: str

# Example scopes
API_SCOPES = {
    "users.read": "Read user profile information",
    "users.write": "Modify user profile information"
}
```

### 2.2 Required Scopes by Service
| Service | Scope | Description |
|---------|-------|-------------|
| [Service Name] | [scope] | [description] |

### 2.3 Scope Implementation
```python
from functools import wraps
from typing import List

def require_scope(required_scope: str):
    async def dependency(token: dict = Depends(validate_token)):
        token_scopes = token.get('scp', '').split(' ')
        if required_scope not in token_scopes:
            raise HTTPException(
                status_code=403,
                detail={
                    "error": "Insufficient scope",
                    "required_scope": required_scope
                }
            )
        return token
    return dependency

# Usage example
@app.get("/api/users/")
async def get_users(token: dict = Depends(require_scope("users.read"))):
    # Implementation
    return {"message": "Users list"}
```

## 3. Implementation Guidelines

### 3.1 Environment Configuration
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    azure_client_id: str
    azure_client_secret: str
    azure_tenant_id: str
    
    class Config:
        env_file = ".env"

# Example .env file
"""
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-client-secret
AZURE_TENANT_ID=your-tenant-id
"""
```

### 3.2 Dependencies
```python
# requirements.txt
fastapi==0.104.1
msal==1.20.0
python-jose[cryptography]==3.3.0
requests==2.28.1
python-dotenv==0.21.0
uvicorn==0.22.0
pydantic==2.4.2
pydantic-settings==2.0.3
```

### 3.3 Security Best Practices
1. Always use HTTPS
2. Store credentials in environment variables
3. Implement proper error handling
4. Log authentication failures
5. Regular security audits

## 4. Testing Requirements

### 4.1 Unit Tests
```python
from fastapi.testclient import TestClient
import pytest
from unittest.mock import patch

client = TestClient(app)

def test_protected_endpoint():
    # Test with valid token
    valid_token = "valid.test.token"
    response = client.get(
        "/api/users",
        headers={"Authorization": f"Bearer {valid_token}"}
    )
    assert response.status_code == 200

    # Test with invalid token
    invalid_token = "invalid.token"
    response = client.get(
        "/api/users",
        headers={"Authorization": f"Bearer {invalid_token}"}
    )
    assert response.status_code == 401
```

### 4.2 Integration Tests
1. Test authentication with valid Azure AD tokens
2. Test scope validation for each endpoint
3. Verify proper error handling for:
   - Invalid tokens
   - Missing scopes
   - Expired tokens

### 4.3 Test Environment
- Use separate Azure AD application registration
- Configure test scopes
- Mock external dependencies
