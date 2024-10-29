# Companies Endpoint

The Companies endpoint allows you to manage company records in Autotask. This includes creating new companies, updating existing ones, and retrieving company information.

## Endpoint Base Path

```
/Companies
```

## Available Methods

### List Companies

```http
GET /Companies
```

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | string | Search companies by name |
| `active` | boolean | Filter by active status |
| `page` | integer | Page number |
| `pageSize` | integer | Results per page |

#### Example Response

```json
{
  "items": [
    {
      "id": 12345,
      "name": "Example Corp",
      "address1": "123 Main St",
      "city": "Boston",
      "state": "MA",
      "postalCode": "02108",
      "phone": "555-0123",
      "active": true,
      "createDate": "2024-01-15T10:30:00Z",
      "lastModifiedDate": "2024-01-15T10:30:00Z"
    }
  ],
  "pageDetails": {
    "count": 100,
    "requestCount": 25,
    "prevPage": 1,
    "nextPage": 3
  }
}
```

### Get Single Company

```http
GET /Companies/{id}
```

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | integer | Company ID |

#### Example Response

```json
{
  "id": 12345,
  "name": "Example Corp",
  "address1": "123 Main St",
  "city": "Boston",
  "state": "MA",
  "postalCode": "02108",
  "phone": "555-0123",
  "active": true,
  "createDate": "2024-01-15T10:30:00Z",
  "lastModifiedDate": "2024-01-15T10:30:00Z"
}
```

### Create Company

```http
POST /Companies
```

#### Request Body

```json
{
  "name": "New Company Inc",
  "address1": "456 Business Ave",
  "city": "Chicago",
  "state": "IL",
  "postalCode": "60601",
  "phone": "555-0456",
  "active": true
}
```

#### Required Fields

- `name`
- `address1`
- `city`
- `state`
- `postalCode`

### Update Company

```http
PATCH /Companies/{id}
```

#### Request Body

```json
{
  "phone": "555-9876",
  "address1": "789 Updated St"
}
```

### Delete Company

```http
DELETE /Companies/{id}
```

## Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Unique identifier |
| `name` | string | Company name |
| `address1` | string | Primary address |
| `address2` | string | Secondary address |
| `city` | string | City |
| `state` | string | State/Province |
| `postalCode` | string | Postal/ZIP code |
| `country` | string | Country |
| `phone` | string | Primary phone |
| `fax` | string | Fax number |
| `website` | string | Website URL |
| `active` | boolean | Active status |
| `createDate` | datetime | Creation timestamp |
| `lastModifiedDate` | datetime | Last update timestamp |

## Error Responses

### Company Not Found

```json
{
  "error": {
    "code": "COMPANY_NOT_FOUND",
    "message": "Company with ID 12345 not found",
    "status": 404
  }
}
```

### Validation Error

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid company data",
    "details": {
      "name": "Company name is required",
      "postalCode": "Invalid postal code format"
    },
    "status": 400
  }
}
```

## Code Examples

### Python Example

```python
import requests

def get_company(api_token, company_id):
    headers = {
        'Authorization': f'Bearer {api_token}',
        'Content-Type': 'application/json'
    }
    
    response = requests.get(
        f'https://webservices.autotask.net/ATServicesRest/V1.0/Companies/{company_id}',
        headers=headers
    )
    
    return response.json()

def create_company(api_token, company_data):
    headers = {
        'Authorization': f'Bearer {api_token}',
        'Content-Type': 'application/json'
    }
    
    response = requests.post(
        'https://webservices.autotask.net/ATServicesRest/V1.0/Companies',
        headers=headers,
        json=company_data
    )
    
    return response.json()
```

### JavaScript Example

```javascript
async function getCompany(apiToken, companyId) {
    const response = await fetch(
        `https://webservices.autotask.net/ATServicesRest/V1.0/Companies/${companyId}`,
        {
            headers: {
                'Authorization': `Bearer ${apiToken}`,
                'Content-Type': 'application/json'
            }
        }
    );
    
    return await response.json();
}

async function createCompany(apiToken, companyData) {
    const response = await fetch(
        'https://webservices.autotask.net/ATServicesRest/V1.0/Companies',
        {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${apiToken}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(companyData)
        }
    );
    
    return await response.json();
}
```

## Best Practices

1. **Searching Companies**
   - Use specific search terms
   - Implement pagination for large result sets
   - Cache frequently accessed company data

2. **Creating Companies**
   - Validate data before submission
   - Check for duplicates
   - Include all required fields

3. **Updating Companies**
   - Use PATCH for partial updates
   - Verify changes after update
   - Handle concurrent modifications

4. **Error Handling**
   - Implement proper error handling
   - Log API responses
   - Retry failed requests with backoff

## Rate Limiting

- Standard rate limit applies
- Implement exponential backoff
- Cache frequently accessed data
