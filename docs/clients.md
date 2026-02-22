# Clients API

Manage client contacts who will receive invoice reminders.

## Base Endpoint

```
/api/clients
```

## Client Object

```json
{
  "id": "cljk1x2y3000008l7abc123",
  "account_id": "acc_xyz789",
  "client_name": "ACME Corp",
  "client_email": "billing@acme.com",
  "company_name": "ACME Corporation",
  "notes": "Net 30 payment terms, contact Sarah for questions",
  "created_at": "2024-01-15T10:00:00.000Z",
  "updated_at": "2024-02-10T14:30:00.000Z"
}
```

### Attributes

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier for the client |
| `account_id` | string | Your account ID |
| `client_name` | string | Client's name or business name |
| `client_email` | string | Client's email address (must be unique per account) |
| `company_name` | string (optional) | Full company name |
| `notes` | string (optional) | Internal notes about this client |
| `created_at` | datetime | When the client was created |
| `updated_at` | datetime | When the client was last updated |

## List All Clients

Retrieve all clients for your account.

```bash
GET /api/clients
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number for pagination |
| `limit` | integer | 50 | Number of results per page (max 100) |
| `search` | string | - | Search by name or email |

### Example Request

```bash
curl https://uptillo.com/api/clients?page=1&limit=20 \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "data": [
    {
      "id": "cljk1x2y3000008l7abc123",
      "client_name": "ACME Corp",
      "client_email": "billing@acme.com",
      "company_name": "ACME Corporation",
      "notes": "Net 30 payment terms",
      "created_at": "2024-01-15T10:00:00.000Z",
      "updated_at": "2024-01-15T10:00:00.000Z"
    },
    {
      "id": "cljk1x2y3000008l7def456",
      "client_name": "TechStart Inc",
      "client_email": "accounts@techstart.io",
      "company_name": "TechStart Incorporated",
      "notes": null,
      "created_at": "2024-01-20T12:30:00.000Z",
      "updated_at": "2024-01-20T12:30:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 47,
    "pages": 3
  }
}
```

## Get a Single Client

Retrieve details of a specific client.

```bash
GET /api/clients/:client_id
```

### Example Request

```bash
curl https://uptillo.com/api/clients/cljk1x2y3000008l7abc123 \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "cljk1x2y3000008l7abc123",
    "client_name": "ACME Corp",
    "client_email": "billing@acme.com",
    "company_name": "ACME Corporation",
    "notes": "Net 30 payment terms",
    "created_at": "2024-01-15T10:00:00.000Z",
    "updated_at": "2024-01-15T10:00:00.000Z",
    "invoice_chases": [
      {
        "id": "chase_789xyz",
        "invoice_number": "INV-2024-001",
        "status": "ACTIVE",
        "emails_sent_count": 2
      }
    ]
  }
}
```

## Create a Client

Add a new client to your account.

```bash
POST /api/clients
```

### Request Body

```json
{
  "client_name": "ACME Corp",
  "client_email": "billing@acme.com",
  "company_name": "ACME Corporation",
  "notes": "Net 30 payment terms"
}
```

### Required Fields

- `client_name` - Client's name or business name
- `client_email` - Valid email address (must be unique)

### Example Request

```bash
curl https://uptillo.com/api/clients \
  -H "Authorization: Bearer pk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "client_name": "ACME Corp",
    "client_email": "billing@acme.com",
    "company_name": "ACME Corporation",
    "notes": "Net 30 payment terms"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "cljk1x2y3000008l7abc123",
    "client_name": "ACME Corp",
    "client_email": "billing@acme.com",
    "company_name": "ACME Corporation",
    "notes": "Net 30 payment terms",
    "created_at": "2024-02-10T15:00:00.000Z",
    "updated_at": "2024-02-10T15:00:00.000Z"
  }
}
```

## Update a Client

Modify an existing client's information.

```bash
PATCH /api/clients/:client_id
```

### Request Body

```json
{
  "client_name": "ACME Corporation Ltd",
  "notes": "Updated payment terms: Net 15"
}
```

### Example Request

```bash
curl -X PATCH https://uptillo.com/api/clients/cljk1x2y3000008l7abc123 \
  -H "Authorization: Bearer pk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "client_name": "ACME Corporation Ltd",
    "notes": "Updated payment terms: Net 15"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "cljk1x2y3000008l7abc123",
    "client_name": "ACME Corporation Ltd",
    "client_email": "billing@acme.com",
    "company_name": "ACME Corporation",
    "notes": "Updated payment terms: Net 15",
    "created_at": "2024-01-15T10:00:00.000Z",
    "updated_at": "2024-02-10T15:30:00.000Z"
  }
}
```

## Delete a Client

Remove a client from your account.

**Warning**: This will also delete all associated invoice chases and email logs.

```bash
DELETE /api/clients/:client_id
```

### Example Request

```bash
curl -X DELETE https://uptillo.com/api/clients/cljk1x2y3000008l7abc123 \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "message": "Client deleted successfully"
}
```

## Search Clients

Search for clients by name or email.

```bash
GET /api/clients?search=acme
```

### Example Request

```bash
curl "https://uptillo.com/api/clients?search=acme" \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "data": [
    {
      "id": "cljk1x2y3000008l7abc123",
      "client_name": "ACME Corp",
      "client_email": "billing@acme.com",
      "company_name": "ACME Corporation",
      "notes": "Net 30 payment terms",
      "created_at": "2024-01-15T10:00:00.000Z",
      "updated_at": "2024-01-15T10:00:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 1,
    "pages": 1
  }
}
```

## Check Opt-Out Status

Check if a client has opted out of receiving reminders.

```bash
GET /api/clients/:client_id/opt-out-status
```

### Example Response

```json
{
  "success": true,
  "data": {
    "opted_out": true,
    "opted_out_at": "2024-02-05T10:30:00.000Z",
    "reason": "Already paid all invoices"
  }
}
```

## Error Responses

### Duplicate Email

```json
{
  "success": false,
  "error": {
    "code": "DUPLICATE_CLIENT",
    "message": "A client with this email already exists",
    "details": {
      "field": "client_email",
      "existing_client_id": "cljk1x2y3000008l7abc123"
    }
  }
}
```

HTTP Status: `409 Conflict`

### Client Not Found

```json
{
  "success": false,
  "error": {
    "code": "CLIENT_NOT_FOUND",
    "message": "No client found with the provided ID"
  }
}
```

HTTP Status: `404 Not Found`

### Validation Error

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": {
      "field": "client_email",
      "value": "invalid-email"
    }
  }
}
```

HTTP Status: `400 Bad Request`

## Code Examples

### JavaScript (Node.js)

```javascript
const axios = require('axios');

const apiKey = process.env.UPTILLO_API_KEY;
const baseURL = 'https://uptillo.com/api';

// Create a client
async function createClient() {
  try {
    const response = await axios.post(`${baseURL}/clients`, {
      client_name: 'ACME Corp',
      client_email: 'billing@acme.com',
      company_name: 'ACME Corporation',
      notes: 'Net 30 payment terms'
    }, {
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      }
    });

    console.log('Client created:', response.data.data);
    return response.data.data;
  } catch (error) {
    console.error('Error:', error.response.data);
  }
}

// List all clients
async function listClients() {
  try {
    const response = await axios.get(`${baseURL}/clients`, {
      headers: {
        'Authorization': `Bearer ${apiKey}`
      }
    });

    console.log(`Found ${response.data.data.length} clients`);
    return response.data.data;
  } catch (error) {
    console.error('Error:', error.response.data);
  }
}
```

### Python

```python
import requests
import os

api_key = os.getenv('UPTILLO_API_KEY')
base_url = 'https://uptillo.com/api'

# Create a client
def create_client():
    headers = {
        'Authorization': f'Bearer {api_key}',
        'Content-Type': 'application/json'
    }

    data = {
        'client_name': 'ACME Corp',
        'client_email': 'billing@acme.com',
        'company_name': 'ACME Corporation',
        'notes': 'Net 30 payment terms'
    }

    response = requests.post(f'{base_url}/clients', json=data, headers=headers)

    if response.status_code == 200:
        client = response.json()['data']
        print(f"Client created: {client['id']}")
        return client
    else:
        print(f"Error: {response.json()}")

# List all clients
def list_clients():
    headers = {
        'Authorization': f'Bearer {api_key}'
    }

    response = requests.get(f'{base_url}/clients', headers=headers)

    if response.status_code == 200:
        clients = response.json()['data']
        print(f"Found {len(clients)} clients")
        return clients
    else:
        print(f"Error: {response.json()}")
```

## Next Steps

- [Invoice Chases API](invoice-chases.md)
- [Email Templates API](templates.md)
- [Webhooks](webhooks.md)
