# Invoice Chases API

Automate invoice reminder workflows with the Invoice Chases API.

## Base Endpoint

```
/api/invoice-chases
```

## Invoice Chase Object

```json
{
  "id": "chase_abc123xyz",
  "account_id": "acc_xyz789",
  "client_id": "client_def456",
  "invoice_number": "INV-2024-001",
  "invoice_amount": "1250.00",
  "currency": "EUR",
  "due_date": "2024-03-15T00:00:00.000Z",
  "invoice_date": "2024-02-14T00:00:00.000Z",
  "status": "ACTIVE",
  "next_send_at": "2024-03-18T09:00:00.000Z",
  "emails_sent_count": 2,
  "custom_note": "Please reference PO #12345 when paying",
  "payment_link": "https://pay.example.com/invoice/001",
  "needs_attention": false,
  "payment_scheduled_for": null,
  "response_state": null,
  "created_at": "2024-02-14T10:00:00.000Z",
  "updated_at": "2024-03-16T14:30:00.000Z"
}
```

### Attributes

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier for the invoice chase |
| `account_id` | string | Your account ID |
| `client_id` | string | ID of the client who will receive reminders |
| `invoice_number` | string | Your invoice number (e.g., "INV-2024-001") |
| `invoice_amount` | decimal | Invoice amount (up to 10 digits, 2 decimals) |
| `currency` | string | Currency code (EUR, USD, GBP, etc.) |
| `due_date` | datetime | When the invoice is due |
| `invoice_date` | datetime (optional) | When the invoice was issued |
| `status` | string | Current status: ACTIVE, PAUSED, PAID, CANCELLED |
| `next_send_at` | datetime | When the next reminder will be sent |
| `emails_sent_count` | integer | Number of reminders sent |
| `custom_note` | string (optional) | Custom message to include in reminders |
| `payment_link` | string (optional) | URL where client can pay |
| `needs_attention` | boolean | True if client responded with an issue |
| `payment_scheduled_for` | datetime (optional) | Date client promised to pay |
| `response_state` | string (optional) | Client's response: WILL_PAY, ALREADY_PAID, DISPUTE |
| `created_at` | datetime | When the chase was created |
| `updated_at` | datetime | When the chase was last updated |

### Status Values

| Status | Description |
|--------|-------------|
| `ACTIVE` | Reminders are being sent automatically |
| `PAUSED` | Reminders are paused (can be resumed) |
| `PAID` | Invoice has been marked as paid |
| `CANCELLED` | Invoice chase was cancelled |

## List All Invoice Chases

Retrieve all invoice chases for your account.

```bash
GET /api/invoice-chases
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number for pagination |
| `limit` | integer | 50 | Number of results per page (max 100) |
| `status` | string | - | Filter by status (ACTIVE, PAUSED, PAID, CANCELLED) |
| `client_id` | string | - | Filter by client ID |

### Example Request

```bash
curl "https://uptillo.com/api/invoice-chases?status=ACTIVE&page=1&limit=20" \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "data": [
    {
      "id": "chase_abc123xyz",
      "client_id": "client_def456",
      "invoice_number": "INV-2024-001",
      "invoice_amount": "1250.00",
      "currency": "EUR",
      "due_date": "2024-03-15T00:00:00.000Z",
      "status": "ACTIVE",
      "next_send_at": "2024-03-18T09:00:00.000Z",
      "emails_sent_count": 2,
      "payment_link": "https://pay.example.com/invoice/001",
      "created_at": "2024-02-14T10:00:00.000Z",
      "client": {
        "id": "client_def456",
        "client_name": "ACME Corp",
        "client_email": "billing@acme.com"
      }
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 15,
    "pages": 1
  }
}
```

## Get a Single Invoice Chase

Retrieve details of a specific invoice chase.

```bash
GET /api/invoice-chases/:chase_id
```

### Example Request

```bash
curl https://uptillo.com/api/invoice-chases/chase_abc123xyz \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "chase_abc123xyz",
    "client_id": "client_def456",
    "invoice_number": "INV-2024-001",
    "invoice_amount": "1250.00",
    "currency": "EUR",
    "due_date": "2024-03-15T00:00:00.000Z",
    "status": "ACTIVE",
    "emails_sent_count": 2,
    "payment_link": "https://pay.example.com/invoice/001",
    "client": {
      "id": "client_def456",
      "client_name": "ACME Corp",
      "client_email": "billing@acme.com",
      "company_name": "ACME Corporation"
    },
    "email_logs": [
      {
        "id": "log_123",
        "template_key": "followup_3_days",
        "sent_at": "2024-03-18T09:00:00.000Z",
        "delivery_status": "delivered",
        "opened_at": "2024-03-18T10:15:00.000Z"
      },
      {
        "id": "log_124",
        "template_key": "followup_7_days",
        "sent_at": "2024-03-22T09:00:00.000Z",
        "delivery_status": "delivered",
        "opened_at": null
      }
    ]
  }
}
```

## Create an Invoice Chase

Start automated reminders for a new invoice.

```bash
POST /api/invoice-chases
```

### Request Body

```json
{
  "client_id": "client_def456",
  "invoice_number": "INV-2024-001",
  "invoice_amount": 1250.00,
  "currency": "EUR",
  "due_date": "2024-03-15",
  "invoice_date": "2024-02-14",
  "custom_note": "Please reference PO #12345 when paying",
  "payment_link": "https://pay.example.com/invoice/001"
}
```

### Required Fields

- `client_id` - ID of an existing client
- `invoice_number` - Your invoice number
- `invoice_amount` - Invoice total (number or string)
- `currency` - 3-letter currency code (EUR, USD, GBP, etc.)
- `due_date` - Invoice due date (YYYY-MM-DD or ISO datetime)

### Example Request

```bash
curl https://uptillo.com/api/invoice-chases \
  -H "Authorization: Bearer pk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "client_id": "client_def456",
    "invoice_number": "INV-2024-001",
    "invoice_amount": 1250.00,
    "currency": "EUR",
    "due_date": "2024-03-15",
    "payment_link": "https://pay.example.com/invoice/001"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "chase_abc123xyz",
    "client_id": "client_def456",
    "invoice_number": "INV-2024-001",
    "invoice_amount": "1250.00",
    "currency": "EUR",
    "due_date": "2024-03-15T00:00:00.000Z",
    "status": "ACTIVE",
    "next_send_at": "2024-03-18T09:00:00.000Z",
    "emails_sent_count": 0,
    "payment_link": "https://pay.example.com/invoice/001",
    "created_at": "2024-02-14T10:00:00.000Z",
    "updated_at": "2024-02-14T10:00:00.000Z"
  }
}
```

## Update an Invoice Chase

Modify an existing invoice chase.

```bash
PATCH /api/invoice-chases/:chase_id
```

### Request Body

```json
{
  "invoice_amount": 1300.00,
  "due_date": "2024-03-20",
  "custom_note": "Updated: Please include PO #12345",
  "payment_link": "https://pay.example.com/invoice/001-updated"
}
```

### Example Request

```bash
curl -X PATCH https://uptillo.com/api/invoice-chases/chase_abc123xyz \
  -H "Authorization: Bearer pk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "invoice_amount": 1300.00,
    "custom_note": "Updated: Please include PO #12345"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "chase_abc123xyz",
    "invoice_number": "INV-2024-001",
    "invoice_amount": "1300.00",
    "custom_note": "Updated: Please include PO #12345",
    "updated_at": "2024-02-14T11:00:00.000Z"
  }
}
```

## Pause an Invoice Chase

Temporarily stop sending reminders.

```bash
POST /api/invoice-chases/:chase_id/pause
```

### Example Request

```bash
curl -X POST https://uptillo.com/api/invoice-chases/chase_abc123xyz/pause \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "chase_abc123xyz",
    "status": "PAUSED",
    "next_send_at": null
  }
}
```

## Resume an Invoice Chase

Resume sending reminders after pausing.

```bash
POST /api/invoice-chases/:chase_id/resume
```

### Example Request

```bash
curl -X POST https://uptillo.com/api/invoice-chases/chase_abc123xyz/resume \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "chase_abc123xyz",
    "status": "ACTIVE",
    "next_send_at": "2024-03-18T09:00:00.000Z"
  }
}
```

## Mark as Paid

Mark an invoice as paid and stop reminders.

```bash
POST /api/invoice-chases/:chase_id/mark-paid
```

### Request Body (Optional)

```json
{
  "payment_date": "2024-03-16"
}
```

### Example Request

```bash
curl -X POST https://uptillo.com/api/invoice-chases/chase_abc123xyz/mark-paid \
  -H "Authorization: Bearer pk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_date": "2024-03-16"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "chase_abc123xyz",
    "status": "PAID",
    "next_send_at": null
  },
  "message": "Invoice marked as paid"
}
```

## Cancel an Invoice Chase

Cancel an invoice chase permanently.

```bash
DELETE /api/invoice-chases/:chase_id
```

### Example Request

```bash
curl -X DELETE https://uptillo.com/api/invoice-chases/chase_abc123xyz \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "message": "Invoice chase cancelled successfully"
}
```

## Get Email History

Retrieve all emails sent for a specific invoice chase.

```bash
GET /api/invoice-chases/:chase_id/emails
```

### Example Response

```json
{
  "success": true,
  "data": [
    {
      "id": "log_123",
      "template_key": "followup_3_days",
      "recipient_email": "billing@acme.com",
      "subject": "Friendly reminder: Invoice INV-2024-001 is 3 days overdue",
      "sent_at": "2024-03-18T09:00:00.000Z",
      "delivery_status": "delivered",
      "opened_at": "2024-03-18T10:15:00.000Z",
      "clicked_at": "2024-03-18T10:16:00.000Z"
    },
    {
      "id": "log_124",
      "template_key": "followup_7_days",
      "recipient_email": "billing@acme.com",
      "subject": "Important: Invoice INV-2024-001 is 7 days overdue",
      "sent_at": "2024-03-22T09:00:00.000Z",
      "delivery_status": "delivered",
      "opened_at": null,
      "clicked_at": null
    }
  ]
}
```

## Reminder Schedule

Uptillo automatically sends reminders at these intervals after the due date:

| Days Overdue | Template | Subject Line |
|--------------|----------|--------------|
| 3 days | `followup_3_days` | "Friendly reminder: Invoice {number} is overdue" |
| 7 days | `followup_7_days` | "Important: Invoice {number} is 7 days overdue" |
| 14 days | `followup_14_days` | "Final reminder: Invoice {number} is 14 days overdue" |

All reminders are sent at 9:00 AM in your account's timezone.

## Error Responses

### Invalid Client ID

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

### Duplicate Invoice Number

```json
{
  "success": false,
  "error": {
    "code": "DUPLICATE_INVOICE",
    "message": "An active chase already exists for this invoice number",
    "details": {
      "field": "invoice_number",
      "existing_chase_id": "chase_existing123"
    }
  }
}
```

HTTP Status: `409 Conflict`

### Validation Error

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "invoice_amount must be greater than 0",
    "details": {
      "field": "invoice_amount",
      "value": -100
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

// Create an invoice chase
async function createInvoiceChase(clientId) {
  try {
    const response = await axios.post(`${baseURL}/invoice-chases`, {
      client_id: clientId,
      invoice_number: 'INV-2024-001',
      invoice_amount: 1250.00,
      currency: 'EUR',
      due_date: '2024-03-15',
      payment_link: 'https://pay.example.com/invoice/001'
    }, {
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json'
      }
    });

    console.log('Invoice chase created:', response.data.data.id);
    return response.data.data;
  } catch (error) {
    console.error('Error:', error.response.data);
  }
}

// Mark as paid
async function markAsPaid(chaseId) {
  try {
    const response = await axios.post(
      `${baseURL}/invoice-chases/${chaseId}/mark-paid`,
      { payment_date: new Date().toISOString().split('T')[0] },
      {
        headers: {
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/json'
        }
      }
    );

    console.log('Marked as paid:', response.data.message);
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
from datetime import datetime, timedelta

api_key = os.getenv('UPTILLO_API_KEY')
base_url = 'https://uptillo.com/api'

# Create an invoice chase
def create_invoice_chase(client_id):
    headers = {
        'Authorization': f'Bearer {api_key}',
        'Content-Type': 'application/json'
    }

    due_date = (datetime.now() + timedelta(days=30)).strftime('%Y-%m-%d')

    data = {
        'client_id': client_id,
        'invoice_number': 'INV-2024-001',
        'invoice_amount': 1250.00,
        'currency': 'EUR',
        'due_date': due_date,
        'payment_link': 'https://pay.example.com/invoice/001'
    }

    response = requests.post(f'{base_url}/invoice-chases', json=data, headers=headers)

    if response.status_code == 200:
        chase = response.json()['data']
        print(f"Invoice chase created: {chase['id']}")
        return chase
    else:
        print(f"Error: {response.json()}")

# Mark as paid
def mark_as_paid(chase_id):
    headers = {
        'Authorization': f'Bearer {api_key}',
        'Content-Type': 'application/json'
    }

    data = {
        'payment_date': datetime.now().strftime('%Y-%m-%d')
    }

    response = requests.post(
        f'{base_url}/invoice-chases/{chase_id}/mark-paid',
        json=data,
        headers=headers
    )

    if response.status_code == 200:
        print(f"Marked as paid: {response.json()['message']}")
        return response.json()['data']
    else:
        print(f"Error: {response.json()}")
```

## Next Steps

- [Email Templates API](templates.md)
- [Webhooks](webhooks.md)
- [Client Responses](webhooks.md#client-response-events)
