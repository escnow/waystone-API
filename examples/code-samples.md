# Code Samples

This section provides practical examples of common operations using the Autotask API in various programming languages.

## Authentication

### Python

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

class AutotaskAPI:
    def __init__(self, client_id, client_secret):
        self.token = get_access_token(client_id, client_secret)
        self.headers = {
            "Authorization": f"Bearer {self.token}",
            "Content-Type": "application/json"
        }
```

### JavaScript

```javascript
async function getAccessToken(clientId, clientSecret) {
    const url = 'https://webservices.autotask.net/ATServicesRest/oauth/token';
    const params = new URLSearchParams();
    params.append('grant_type', 'client_credentials');
    params.append('client_id', clientId);
    params.append('client_secret', clientSecret);
    
    const response = await fetch(url, {
        method: 'POST',
        body: params
    });
    
    const data = await response.json();
    return data.access_token;
}

class AutotaskAPI {
    constructor(clientId, clientSecret) {
        this.initialize(clientId, clientSecret);
    }
    
    async initialize(clientId, clientSecret) {
        this.token = await getAccessToken(clientId, clientSecret);
        this.headers = {
            'Authorization': `Bearer ${this.token}`,
            'Content-Type': 'application/json'
        };
    }
}
```

## Company Operations

### Create Company

```python
def create_company(api, company_data):
    url = "https://webservices.autotask.net/ATServicesRest/V1.0/Companies"
    response = requests.post(url, headers=api.headers, json=company_data)
    return response.json()

# Usage example
company_data = {
    "name": "New Company Inc",
    "address1": "123 Business St",
    "city": "Boston",
    "state": "MA",
    "postalCode": "02108"
}

new_company = create_company(api, company_data)
```

### Update Company

```javascript
async function updateCompany(api, companyId, updateData) {
    const url = `https://webservices.autotask.net/ATServicesRest/V1.0/Companies/${companyId}`;
    const response = await fetch(url, {
        method: 'PATCH',
        headers: api.headers,
        body: JSON.stringify(updateData)
    });
    
    return await response.json();
}

// Usage example
const updates = {
    phone: '555-0123',
    website: 'https://example.com'
};

const updatedCompany = await updateCompany(api, 12345, updates);
```

## Ticket Management

### Create Ticket

```python
def create_ticket(api, ticket_data):
    url = "https://webservices.autotask.net/ATServicesRest/V1.0/Tickets"
    response = requests.post(url, headers=api.headers, json=ticket_data)
    return response.json()

# Usage example
ticket_data = {
    "title": "Server Down",
    "description": "Production server is not responding",
    "priority": 1,
    "status": 1,
    "queueID": 12345,
    "companyID": 67890
}

new_ticket = create_ticket(api, ticket_data)
```

### Update Ticket Status

```javascript
async function updateTicketStatus(api, ticketId, status) {
    const url = `https://webservices.autotask.net/ATServicesRest/V1.0/Tickets/${ticketId}`;
    const response = await fetch(url, {
        method: 'PATCH',
        headers: api.headers,
        body: JSON.stringify({ status })
    });
    
    return await response.json();
}

// Usage example
await updateTicketStatus(api, 12345, 5); // Mark as completed
```

## Contact Management

### Search Contacts

```python
def search_contacts(api, company_id=None, email=None):
    url = "https://webservices.autotask.net/ATServicesRest/V1.0/Contacts"
    params = {}
    
    if company_id:
        params['filter'] = f"companyID eq {company_id}"
    if email:
        params['filter'] = f"emailAddress eq '{email}'"
        
    response = requests.get(url, headers=api.headers, params=params)
    return response.json()

# Usage example
contacts = search_contacts(api, company_id=12345)
```

### Create Contact

```javascript
async function createContact(api, contactData) {
    const url = 'https://webservices.autotask.net/ATServicesRest/V1.0/Contacts';
    const response = await fetch(url, {
        method: 'POST',
        headers: api.headers,
        body: JSON.stringify(contactData)
    });
    
    return await response.json();
}

// Usage example
const contact = {
    firstName: 'John',
    lastName: 'Doe',
    emailAddress: 'john.doe@example.com',
    companyID: 12345,
    isActive: true
};

const newContact = await createContact(api, contact);
```

## Resource Management

### Get Available Resources

```python
def get_available_resources(api, start_date, end_date):
    url = "https://webservices.autotask.net/ATServicesRest/V1.0/Resources"
    params = {
        'filter': f"active eq true and startDate ge '{start_date}' and endDate le '{end_date}'"
    }
    
    response = requests.get(url, headers=api.headers, params=params)
    return response.json()

# Usage example
from datetime import datetime, timedelta

start = datetime.now()
end = start + timedelta(days=7)
available_resources = get_available_resources(
    api,
    start.strftime('%Y-%m-%d'),
    end.strftime('%Y-%m-%d')
)
```

### Assign Resource to Ticket

```javascript
async function assignResource(api, ticketId, resourceId) {
    const url = `https://webservices.autotask.net/ATServicesRest/V1.0/Tickets/${ticketId}`;
    const response = await fetch(url, {
        method: 'PATCH',
        headers: api.headers,
        body: JSON.stringify({
            resourceID: resourceId,
            status: 2 // In Progress
        })
    });
    
    return await response.json();
}

// Usage example
await assignResource(api, 12345, 67890);
```

## Error Handling Examples

### Python

```python
class AutotaskError(Exception):
    def __init__(self, message, status_code=None, error_data=None):
        self.message = message
        self.status_code = status_code
        self.error_data = error_data
        super().__init__(self.message)

def safe_api_call(func):
