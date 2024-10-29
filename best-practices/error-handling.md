# Error Handling

This guide covers best practices for handling errors in the Autotask API, including common error types, proper error handling patterns, and implementation examples.

## Error Response Format

All API errors follow a consistent format:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": {
      "field1": "Specific error detail",
      "field2": "Another error detail"
    },
    "requestId": "unique-request-id"
  }
}
```

## Common Error Types

### 1. Authentication Errors (401)

```json
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "The access token provided is invalid or has expired",
    "requestId": "auth-123"
  }
}
```

### 2. Authorization Errors (403)

```json
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "User does not have permission to access this resource",
    "requestId": "auth-456"
  }
}
```

### 3. Validation Errors (400)

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid data",
    "details": {
      "email": "Invalid email format",
      "phone": "Phone number is required"
    },
    "requestId": "val-789"
  }
}
```

## Error Handling Best Practices

### 1. Implement Proper Error Classes

```python
class AutotaskAPIError(Exception):
    def __init__(self, code, message, details=None, request_id=None):
        self.code = code
        self.message = message
        self.details = details or {}
        self.request_id = request_id
        super().__init__(self.message)

class ValidationError(AutotaskAPIError):
    pass

class AuthenticationError(AutotaskAPIError):
    pass

class RateLimitError(AutotaskAPIError):
    pass
```

### 2. Handle Errors Gracefully

```python
def handle_api_request(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except AuthenticationError:
            # Handle authentication errors
            refresh_token()
            return func(*args, **kwargs)
        except ValidationError as e:
            # Log validation errors
            logger.error(f"Validation error: {e.details}")
            raise
        except RateLimitError:
            # Implement backoff strategy
            return retry_with_backoff(func, *args, **kwargs)
        except Exception as e:
            # Log unexpected errors
            logger.error(f"Unexpected error: {str(e)}")
            raise
    return wrapper
```

### 3. Implement Retry Logic

```python
def retry_with_backoff(func, *args, max_retries=3, base_delay=1, **kwargs):
    for attempt in range(max_retries):
        try:
            return func(*args, **kwargs)
        except RateLimitError:
            if attempt == max_retries - 1:
                raise
            delay = base_delay * (2 ** attempt)
            time.sleep(delay)
        except (AuthenticationError, ValidationError):
            raise
```

### 4. Error Logging

```python
import logging

class APIErrorLogger:
    def __init__(self):
        self.logger = logging.getLogger('autotask_api')
        self.logger.setLevel(logging.INFO)
        
        # Add handlers
        handler = logging.FileHandler('api_errors.log')
        handler.setFormatter(logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s'
        ))
        self.logger.addHandler(handler)

    def log_error(self, error):
        error_data = {
            'code': error.code,
            'message': error.message,
            'request_id': error.request_id,
            'details': error.details
        }
        self.logger.error(f"API Error: {json.dumps(error_data)}")
```

## Implementation Examples

### Python Example

```python
class AutotaskClient:
    def __init__(self, api_token):
        self.api_token = api_token
        self.error_logger = APIErrorLogger()

    @handle_api_request
    def get_company(self, company_id):
        try:
            response = requests.get(
                f'{BASE_URL}/Companies/{company_id}',
                headers={'Authorization': f'Bearer {self.api_token}'}
            )
            
            if response.status_code == 200:
                return response.json()
                
            error_data = response.json().get('error', {})
            
            if response.status_code == 401:
                raise AuthenticationError(
                    error_data.get('code'),
                    error_data.get('message'),
                    error_data.get('details'),
                    error_data.get('requestId')
                )
            elif response.status_code == 400:
                raise ValidationError(
                    error_data.get('code'),
                    error_data.get('message'),
                    error_data.get('details'),
                    error_data.get('requestId')
                )
            elif response.status_code == 429:
                raise RateLimitError(
                    error_data.get('code'),
                    error_data.get('message'),
                    error_data.get('details'),
                    error_data.get('requestId')
                )
                
        except requests.exceptions.RequestException as e:
            raise AutotaskAPIError(
                'NETWORK_ERROR',
                f'Network error occurred: {str(e)}'
            )
```

### JavaScript Example

```javascript
class AutotaskError extends Error {
    constructor(code, message, details = null, requestId = null) {
        super(message);
        this.code = code;
        this.details = details;
        this.requestId = requestId;
        this.name = this.constructor.name;
    }
}

class AutotaskClient {
    constructor(apiToken) {
        this.apiToken = apiToken;
    }

    async getCompany(companyId) {
        try {
            const response = await fetch(
                `${BASE_URL}/Companies/${companyId}`,
                {
                    headers: {
                        'Authorization': `Bearer ${this.apiToken}`
                    }
                }
            );

            if (response.ok) {
                return await response.json();
            }

            const errorData = await response.json();
            const error = errorData.error;

            switch (response.status) {
                case 401:
                    throw new AuthenticationError(
                        error.code,
                        error.message,
                        error.details,
                        error.requestId
                    );
                case 400:
                    throw new ValidationError(
                        error.code,
                        error.message,
                        error.details,
                        error.requestId
                    );
                case 429:
                    throw new RateLimitError(
                        error.code,
                        error.message,
                        error.details,
                        error.requestId
                    );
                default:
                    throw new AutotaskError(
                        error.code,
                        error.message,
                        error.details,
                        error.requestId
                    );
            }
        } catch (error) {
            if (error instanceof AutotaskError) {
                throw error;
            }
            throw new AutotaskError(
                'NETWORK_ERROR',
                `Network error occurred: ${error.message}`
            );
        }
    }
}
```

## Common Error Codes

| Code | Description | HTTP Status |
|------|-------------|-------------|
| `INVALID_TOKEN` | Invalid or expired token | 401 |
| `INSUFFICIENT_PERMISSIONS` | User lacks required permissions | 403 |
| `VALIDATION_ERROR` | Request data validation failed | 400 |
| `RATE_LIMIT_EXCEEDED` | API rate limit exceeded | 429 |
| `RESOURCE_NOT_FOUND` | Requested resource not found | 404 |
| `INTERNAL_ERROR` | Internal server error | 500 |

## Error Prevention Tips

1. **Validate Data Before Sending**
   - Check required fields
   - Validate data formats
   - Sanitize input

2. **Monitor Token Expiration**
   - Track token expiration time
   - Implement automatic token refresh
   - Handle expired token errors

3. **Implement Circuit Breakers**
   - Monitor error rates
   - Prevent cascade failures
   - Auto-disable problematic features

4. **Use Request IDs**
   - Log request IDs
   - Include in error reports
   - Track request lifecycle

5. **Document Error Handling**
   - Maintain error code documentation
   - Update handling procedures
   - Share best practices
