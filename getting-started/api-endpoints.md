# API Endpoints Overview

This document provides a comprehensive overview of the available Autotask API endpoints and their functionality.

## Base URL

All API requests should be made to:
```
https://webservices.autotask.net/ATServicesRest/V1.0/
```

## Available Endpoints

### Companies

| Endpoint | Method | Description |
|----------|---------|-------------|
| `/Companies` | GET | List all companies |
| `/Companies/{id}` | GET | Get company details |
| `/Companies` | POST | Create new company |
| `/Companies/{id}` | PATCH | Update company |
| `/Companies/{id}` | DELETE | Delete company |

### Contacts

| Endpoint | Method | Description |
|----------|---------|-------------|
| `/Contacts` | GET | List all contacts |
| `/Contacts/{id}` | GET | Get contact details |
| `/Contacts` | POST | Create new contact |
| `/Contacts/{id}` | PATCH | Update contact |
| `/Contacts/{id}` | DELETE | Delete contact |

### Tickets

| Endpoint | Method | Description |
|----------|---------|-------------|
| `/Tickets` | GET | List all tickets |
| `/Tickets/{id}` | GET | Get ticket details |
| `/Tickets` | POST | Create new ticket |
| `/Tickets/{id}` | PATCH | Update ticket |
| `/Tickets/{id}` | DELETE | Delete ticket |

### Resources

| Endpoint | Method | Description |
|----------|---------|-------------|
| `/Resources` | GET | List all resources |
| `/Resources/{id}` | GET | Get resource details |
| `/Resources` | POST | Create new resource |
| `/Resources/{id}` | PATCH | Update resource |
| `/Resources/{id}` | DELETE | Delete resource |

## Query Parameters

Common query parameters available for most endpoints:

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | string | Search term for filtering results |
| `page` | integer | Page number for pagination |
| `pageSize` | integer | Number of items per page |
| `filter` | string | Advanced filtering criteria |
| `sort` | string | Field to sort by |
| `order` | string | Sort order (asc/desc) |

## Response Format

All endpoints return data in JSON format. A typical response structure:

```json
{
  "items": [...],
  "pageDetails": {
    "count": 100,
    "requestCount": 25,
    "prevPage": 1,
    "nextPage": 3
  }
}
```

## Error Responses

Standard error response format:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Detailed error message",
    "details": {
      "field": "Additional information"
    }
  }
}
```

Common HTTP Status Codes:

| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 429 | Too Many Requests |
| 500 | Internal Server Error |

## Rate Limiting

- Default rate limit: 600 requests per minute
- Rate limit headers included in response:
  - `X-RateLimit-Limit`
  - `X-RateLimit-Remaining`
  - `X-RateLimit-Reset`

## Filtering

Example filter syntax:

```
/Companies?filter=name eq 'Example Corp' and active eq true
```

Supported operators:
- `eq` (equals)
- `ne` (not equals)
- `gt` (greater than)
- `lt` (less than)
- `ge` (greater than or equal)
- `le` (less than or equal)
- `contains`
- `startswith`
- `endswith`

## Pagination

Example pagination request:

```
/Tickets?page=2&pageSize=25
```

Default values:
- `page`: 1
- `pageSize`: 25
- Maximum `pageSize`: 100

## Field Selection

Specify fields to return:

```
/Companies?fields=id,name,address
```

## Sorting

Example sorting:

```
/Contacts?sort=lastName&order=desc
```

## Versioning

- Current version: V1.0
- Version included in URL path
- Breaking changes will increment version number
