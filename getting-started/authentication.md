# Authentication

The Autotask API uses OAuth 2.0 for authentication. This guide explains how to authenticate your requests and maintain secure access to the API.

## OAuth 2.0 Flow

### 1. Obtain Credentials

First, you'll need to obtain your API credentials:

1. Log in to Autotask
2. Navigate to Admin > API User Security
3. Create a new API Integration
4. Save your Client ID and Client Secret

### 2. Generate Access Token

```http
POST https://webservices.autotask.net/ATServicesRest/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
client_id=YOUR_CLIENT_ID
client_secret=YOUR_CLIENT_SECRET
```

### 3. Response Format

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## Using the Access Token

Include the access token in all API requests:

```http
GET https://webservices.autotask.net/ATServicesRest/V1.0/Companies
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6...
```

## Token Lifecycle

- Tokens expire after 1 hour (3600 seconds)
- Generate a new token when the current one expires
- Don't share tokens between applications
- Store tokens securely

## Security Best Practices

1. **Secure Storage**
   - Never expose credentials in client-side code
   - Use secure environment variables
   - Implement proper key management

2. **Token Management**
   - Implement token refresh logic
   - Handle token expiration gracefully
   - Revoke compromised tokens immediately

3. **Request Security**
   - Always use HTTPS
   - Validate SSL certificates
   - Implement request signing when required

## Error Handling

Common authentication errors:

| Status Code | Description | Solution |
|------------|-------------|----------|
| 401 | Unauthorized | Invalid or expired token |
| 403 | Forbidden | Insufficient permissions |
| 400 | Bad Request | Invalid client credentials |

Example error response:

```json
{
  "error": "invalid_token",
  "error_description": "The access token has expired"
}
```

## Rate Limiting

Authentication-related endpoints have specific rate limits:

- Token generation: 10 requests per minute
- API calls: 600 requests per minute per token

## Code Examples

### Python Example

```python
import requests

def get_access_token(client_id, client_secret):
    url = "https://webservices.autotask.net/ATServicesRest/oauth/token"
    data = {
        "grant_type": "client_credentials",
        "client_id": client_id,
        "client_secret": client_secret
    }
    response = requests.post(url, data=data)
    return response.json()["access_token"]
```

### JavaScript Example

```javascript
async function getAccessToken(clientId, clientSecret) {
    const url = 'https://webservices.autotask.net/ATServicesRest/oauth/token';
    const params = new URLSearchParams();
    params.append('grant_type', 'client_credentials');
    params.append('client_id', clientId);
    params.append('client_secret', clientSecret);
    
    const response = await fetch(url, {
        method: 'POST',
        body: params,
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        }
    });
    
    const data = await response.json();
    return data.access_token;
}
```

## Troubleshooting

1. **Token Generation Failed**
   - Verify credentials are correct
   - Check API access is enabled
   - Confirm network connectivity

2. **Token Expired**
   - Implement automatic token refresh
   - Monitor token expiration
   - Handle refresh errors gracefully

3. **Permission Issues**
   - Review API user permissions
   - Check integration settings
   - Verify resource access rights
