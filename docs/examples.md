# Code Examples

Practical examples for common Uptillo API integrations.

## Table of Contents

- [Getting Started](#getting-started)
- [Client Management](#client-management)
- [Invoice Chase Workflows](#invoice-chase-workflows)
- [Webhook Integration](#webhook-integration)
- [Email Template Customization](#email-template-customization)
- [Bulk Operations](#bulk-operations)
- [Error Handling](#error-handling)

## Getting Started

### Initialize the API Client

#### JavaScript/Node.js

```javascript
const axios = require('axios');

class UptilloAPI {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseURL = 'https://uptillo.com/api';
  }

  async request(method, endpoint, data = null) {
    try {
      const response = await axios({
        method,
        url: `${this.baseURL}${endpoint}`,
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json'
        },
        data
      });

      return response.data;
    } catch (error) {
      if (error.response) {
        throw new Error(`API Error: ${error.response.data.error.message}`);
      }
      throw error;
    }
  }

  // Convenience methods
  get(endpoint) { return this.request('GET', endpoint); }
  post(endpoint, data) { return this.request('POST', endpoint, data); }
  patch(endpoint, data) { return this.request('PATCH', endpoint, data); }
  delete(endpoint) { return this.request('DELETE', endpoint); }
}

// Usage
const uptillo = new UptilloAPI(process.env.UPTILLO_API_KEY);
```

#### Python

```python
import requests
import os

class UptilloAPI:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = 'https://uptillo.com/api'

    def _request(self, method, endpoint, data=None):
        headers = {
            'Authorization': f'Bearer {self.api_key}',
            'Content-Type': 'application/json'
        }

        url = f'{self.base_url}{endpoint}'

        response = requests.request(method, url, json=data, headers=headers)

        if response.status_code >= 400:
            error = response.json()
            raise Exception(f"API Error: {error['error']['message']}")

        return response.json()

    def get(self, endpoint):
        return self._request('GET', endpoint)

    def post(self, endpoint, data):
        return self._request('POST', endpoint, data)

    def patch(self, endpoint, data):
        return self._request('PATCH', endpoint, data)

    def delete(self, endpoint):
        return self._request('DELETE', endpoint)

# Usage
uptillo = UptilloAPI(os.getenv('UPTILLO_API_KEY'))
```

#### PHP

```php
<?php

class UptilloAPI {
    private $apiKey;
    private $baseURL = 'https://uptillo.com/api';

    public function __construct($apiKey) {
        $this->apiKey = $apiKey;
    }

    private function request($method, $endpoint, $data = null) {
        $ch = curl_init($this->baseURL . $endpoint);

        $headers = [
            'Authorization: Bearer ' . $this->apiKey,
            'Content-Type: application/json'
        ];

        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);

        if ($data !== null) {
            curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
        }

        $response = curl_exec($ch);
        $statusCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        $result = json_decode($response, true);

        if ($statusCode >= 400) {
            throw new Exception('API Error: ' . $result['error']['message']);
        }

        return $result;
    }

    public function get($endpoint) {
        return $this->request('GET', $endpoint);
    }

    public function post($endpoint, $data) {
        return $this->request('POST', $endpoint, $data);
    }

    public function patch($endpoint, $data) {
        return $this->request('PATCH', $endpoint, $data);
    }

    public function delete($endpoint) {
        return $this->request('DELETE', $endpoint);
    }
}

// Usage
$uptillo = new UptilloAPI(getenv('UPTILLO_API_KEY'));
?>
```

## Client Management

### Create a New Client

```javascript
async function createClient(name, email, company = null) {
  const client = await uptillo.post('/clients', {
    client_name: name,
    client_email: email,
    company_name: company,
    notes: 'Created via API'
  });

  console.log('Client created:', client.data.id);
  return client.data;
}

// Usage
const newClient = await createClient('ACME Corp', 'billing@acme.com', 'ACME Corporation');
```

### Find or Create Client

```javascript
async function findOrCreateClient(email, name, company = null) {
  try {
    // Try to find existing client
    const clients = await uptillo.get(`/clients?search=${encodeURIComponent(email)}`);

    if (clients.data.length > 0) {
      console.log('Client found:', clients.data[0].id);
      return clients.data[0];
    }

    // Create new client if not found
    return await createClient(name, email, company);

  } catch (error) {
    console.error('Error finding/creating client:', error.message);
    throw error;
  }
}

// Usage
const client = await findOrCreateClient('billing@acme.com', 'ACME Corp', 'ACME Corporation');
```

### Update Client Information

```javascript
async function updateClientNotes(clientId, notes) {
  const updated = await uptillo.patch(`/clients/${clientId}`, {
    notes: notes
  });

  console.log('Client updated:', updated.data.id);
  return updated.data;
}

// Usage
await updateClientNotes('client_abc123', 'Payment terms: Net 30. Contact: Sarah');
```

## Invoice Chase Workflows

### Create Invoice Chase from Your Invoice System

```javascript
async function createInvoiceChase(invoiceData) {
  // Find or create client
  const client = await findOrCreateClient(
    invoiceData.customerEmail,
    invoiceData.customerName,
    invoiceData.companyName
  );

  // Create invoice chase
  const chase = await uptillo.post('/invoice-chases', {
    client_id: client.id,
    invoice_number: invoiceData.invoiceNumber,
    invoice_amount: invoiceData.amount,
    currency: invoiceData.currency || 'EUR',
    due_date: invoiceData.dueDate,
    invoice_date: invoiceData.invoiceDate,
    payment_link: invoiceData.paymentLink,
    custom_note: invoiceData.notes
  });

  console.log('Invoice chase created:', chase.data.id);
  return chase.data;
}

// Usage
const invoice = {
  invoiceNumber: 'INV-2024-001',
  amount: 1250.00,
  currency: 'EUR',
  dueDate: '2024-03-15',
  invoiceDate: '2024-02-14',
  customerName: 'ACME Corp',
  customerEmail: 'billing@acme.com',
  companyName: 'ACME Corporation',
  paymentLink: 'https://pay.example.com/invoice/001',
  notes: 'Please reference PO #12345 when paying'
};

await createInvoiceChase(invoice);
```

### Mark Invoice as Paid

```javascript
async function markInvoiceAsPaid(invoiceNumber, paymentDate = null) {
  // Find the invoice chase
  const chases = await uptillo.get('/invoice-chases');
  const chase = chases.data.find(c => c.invoice_number === invoiceNumber);

  if (!chase) {
    throw new Error(`Invoice chase not found: ${invoiceNumber}`);
  }

  // Mark as paid
  const result = await uptillo.post(`/invoice-chases/${chase.id}/mark-paid`, {
    payment_date: paymentDate || new Date().toISOString().split('T')[0]
  });

  console.log('Invoice marked as paid:', invoiceNumber);
  return result.data;
}

// Usage
await markInvoiceAsPaid('INV-2024-001', '2024-03-16');
```

### Pause Reminders Temporarily

```javascript
async function pauseReminders(invoiceNumber) {
  const chases = await uptillo.get('/invoice-chases');
  const chase = chases.data.find(c => c.invoice_number === invoiceNumber);

  if (!chase) {
    throw new Error(`Invoice chase not found: ${invoiceNumber}`);
  }

  await uptillo.post(`/invoice-chases/${chase.id}/pause`);
  console.log('Reminders paused for:', invoiceNumber);
}

// Usage
await pauseReminders('INV-2024-001');
```

### Get All Overdue Invoices

```javascript
async function getOverdueInvoices() {
  const chases = await uptillo.get('/invoice-chases?status=ACTIVE');

  const overdue = chases.data.filter(chase => {
    const dueDate = new Date(chase.due_date);
    const today = new Date();
    return dueDate < today;
  });

  console.log(`Found ${overdue.length} overdue invoices`);

  return overdue.map(chase => ({
    invoiceNumber: chase.invoice_number,
    amount: chase.invoice_amount,
    dueDate: chase.due_date,
    daysOverdue: Math.floor((new Date() - new Date(chase.due_date)) / (1000 * 60 * 60 * 24)),
    emailsSent: chase.emails_sent_count,
    client: chase.client
  }));
}

// Usage
const overdueInvoices = await getOverdueInvoices();
overdueInvoices.forEach(inv => {
  console.log(`${inv.invoiceNumber}: ${inv.daysOverdue} days overdue, ${inv.emailsSent} emails sent`);
});
```

## Webhook Integration

### Express.js Webhook Handler

```javascript
const express = require('express');
const crypto = require('crypto');

const app = express();

// Use raw body for signature verification
app.use('/webhooks/uptillo', express.raw({ type: 'application/json' }));

function verifyWebhookSignature(payload, signature, secret) {
  const parts = signature.split(',');
  const timestamp = parts.find(p => p.startsWith('t=')).split('=')[1];
  const receivedSig = parts.find(p => p.startsWith('v1=')).split('=')[1];

  // Check timestamp (prevent replay attacks)
  const now = Math.floor(Date.now() / 1000);
  if (Math.abs(now - parseInt(timestamp)) > 300) {
    throw new Error('Webhook timestamp too old');
  }

  // Compute signature
  const signedPayload = `${timestamp}.${payload}`;
  const expectedSig = crypto
    .createHmac('sha256', secret)
    .update(signedPayload)
    .digest('hex');

  // Compare signatures
  if (receivedSig !== expectedSig) {
    throw new Error('Invalid webhook signature');
  }

  return true;
}

app.post('/webhooks/uptillo', async (req, res) => {
  const signature = req.headers['x-uptillo-signature'];
  const webhookSecret = process.env.UPTILLO_WEBHOOK_SECRET;

  try {
    // Verify signature
    verifyWebhookSignature(req.body.toString(), signature, webhookSecret);

    const event = JSON.parse(req.body);

    // Handle event
    await handleWebhookEvent(event);

    res.status(200).json({ received: true });
  } catch (error) {
    console.error('Webhook error:', error.message);
    res.status(400).json({ error: error.message });
  }
});

async function handleWebhookEvent(event) {
  console.log('Received event:', event.event);

  switch (event.event) {
    case 'invoice.response.submitted':
      await handleClientResponse(event.data);
      break;

    case 'email.delivered':
      await handleEmailDelivered(event.data);
      break;

    case 'email.bounced':
      await handleEmailBounced(event.data);
      break;

    case 'invoice_chase.status_changed':
      await handleStatusChanged(event.data);
      break;

    default:
      console.log('Unhandled event type:', event.event);
  }
}

async function handleClientResponse(data) {
  console.log('Client response:', data.response_type);

  if (data.response_type === 'WILL_PAY') {
    console.log(`Client will pay on: ${data.scheduled_date}`);
    // Update your system
  } else if (data.response_type === 'ALREADY_PAID') {
    console.log('Client claims already paid');
    // Check your payment records
  } else if (data.response_type === 'DISPUTE') {
    console.log('Client disputes invoice');
    // Send notification to accounting team
  }
}

async function handleEmailDelivered(data) {
  console.log(`Email delivered: ${data.recipient_email}`);
  // Log delivery in your system
}

async function handleEmailBounced(data) {
  console.log(`Email bounced: ${data.recipient_email}`);
  console.log(`Reason: ${data.error_message}`);

  if (data.bounce_type === 'hard') {
    // Update client email address
    console.log('Hard bounce - email address invalid');
  }
}

async function handleStatusChanged(data) {
  console.log(`Invoice ${data.invoice_number}: ${data.old_status} → ${data.new_status}`);

  if (data.new_status === 'PAID') {
    // Mark invoice as paid in your system
  }
}

app.listen(3000, () => {
  console.log('Webhook server running on port 3000');
});
```

### Python Flask Webhook Handler

```python
from flask import Flask, request, jsonify
import hmac
import hashlib
import time
import json

app = Flask(__name__)

def verify_webhook_signature(payload, signature, secret):
    parts = dict(part.split('=') for part in signature.split(','))
    timestamp = parts['t']
    received_sig = parts['v1']

    # Check timestamp
    now = int(time.time())
    if abs(now - int(timestamp)) > 300:
        raise ValueError('Webhook timestamp too old')

    # Compute signature
    signed_payload = f"{timestamp}.{payload}"
    expected_sig = hmac.new(
        secret.encode(),
        signed_payload.encode(),
        hashlib.sha256
    ).hexdigest()

    # Compare signatures
    if not hmac.compare_digest(received_sig, expected_sig):
        raise ValueError('Invalid webhook signature')

    return True

@app.route('/webhooks/uptillo', methods=['POST'])
def webhook():
    signature = request.headers.get('X-Uptillo-Signature')
    webhook_secret = os.getenv('UPTILLO_WEBHOOK_SECRET')

    try:
        verify_webhook_signature(request.data.decode(), signature, webhook_secret)

        event = request.json

        # Handle event
        handle_webhook_event(event)

        return jsonify({'received': True}), 200
    except ValueError as e:
        print(f'Webhook verification failed: {e}')
        return jsonify({'error': str(e)}), 400

def handle_webhook_event(event):
    print(f"Received event: {event['event']}")

    if event['event'] == 'invoice.response.submitted':
        handle_client_response(event['data'])
    elif event['event'] == 'email.delivered':
        handle_email_delivered(event['data'])
    elif event['event'] == 'email.bounced':
        handle_email_bounced(event['data'])

def handle_client_response(data):
    print(f"Client response: {data['response_type']}")

    if data['response_type'] == 'WILL_PAY':
        print(f"Client will pay on: {data['scheduled_date']}")
    elif data['response_type'] == 'ALREADY_PAID':
        print('Client claims already paid')
    elif data['response_type'] == 'DISPUTE':
        print('Client disputes invoice')

if __name__ == '__main__':
    app.run(port=3000)
```

## Email Template Customization

### Create Custom Template

```javascript
async function createCustomTemplate() {
  const template = await uptillo.post('/templates', {
    name: 'Friendly 3-Day Reminder',
    subject: 'Quick check-in about invoice {{invoice_number}}',
    body_text: `Hi {{client_name}},

I hope this message finds you well!

I wanted to reach out about invoice {{invoice_number}} for {{invoice_amount}}. According to our records, it was due on {{due_date}}.

If you've already sent payment, please disregard this message. Otherwise, you can pay conveniently online here: {{payment_link}}

Let me know if you have any questions!

Best regards,
{{business_name}}`,
    body_html: `
      <p>Hi {{client_name}},</p>
      <p>I hope this message finds you well!</p>
      <p>I wanted to reach out about invoice <strong>{{invoice_number}}</strong> for <strong>{{invoice_amount}}</strong>. According to our records, it was due on {{due_date}}.</p>
      <p>If you've already sent payment, please disregard this message. Otherwise, you can pay conveniently online here:</p>
      <p><a href="{{payment_link}}" style="background-color: #4F46E5; color: white; padding: 12px 24px; text-decoration: none; border-radius: 6px; display: inline-block;">Pay Now</a></p>
      <p>Let me know if you have any questions!</p>
      <p>Best regards,<br>{{business_name}}</p>
    `,
    template_type: 'followup_3_days',
    is_active: true
  });

  console.log('Template created:', template.data.id);

  // Set as default
  await uptillo.post(`/templates/${template.data.id}/set-default`);

  return template.data;
}
```

### Preview Template with Sample Data

```javascript
async function previewTemplate(templateId) {
  const preview = await uptillo.post(`/templates/${templateId}/preview`, {
    client_name: 'ACME Corp',
    invoice_number: 'INV-2024-001',
    invoice_amount: 1250.00,
    currency: 'EUR',
    due_date: '2024-03-15',
    business_name: 'Your Company Inc',
    payment_link: 'https://pay.example.com/invoice/001'
  });

  console.log('Subject:', preview.data.subject);
  console.log('Body:', preview.data.body_text);

  return preview.data;
}
```

## Bulk Operations

### Bulk Create Invoice Chases

```javascript
async function bulkCreateInvoiceChases(invoices) {
  const results = {
    success: [],
    failed: []
  };

  for (const invoice of invoices) {
    try {
      const chase = await createInvoiceChase(invoice);
      results.success.push({ invoice: invoice.invoiceNumber, chaseId: chase.id });
      console.log(`✓ Created chase for ${invoice.invoiceNumber}`);

      // Rate limiting: wait 100ms between requests
      await new Promise(resolve => setTimeout(resolve, 100));

    } catch (error) {
      results.failed.push({ invoice: invoice.invoiceNumber, error: error.message });
      console.error(`✗ Failed to create chase for ${invoice.invoiceNumber}: ${error.message}`);
    }
  }

  console.log(`\nCompleted: ${results.success.length} success, ${results.failed.length} failed`);

  return results;
}

// Usage
const invoices = [
  { invoiceNumber: 'INV-2024-001', amount: 1250, currency: 'EUR', dueDate: '2024-03-15', customerName: 'ACME Corp', customerEmail: 'billing@acme.com' },
  { invoiceNumber: 'INV-2024-002', amount: 890, currency: 'EUR', dueDate: '2024-03-16', customerName: 'TechStart', customerEmail: 'accounts@techstart.io' },
  // ... more invoices
];

const results = await bulkCreateInvoiceChases(invoices);
```

### Export All Clients

```javascript
async function exportAllClients() {
  let allClients = [];
  let page = 1;
  const limit = 100;

  while (true) {
    const response = await uptillo.get(`/clients?page=${page}&limit=${limit}`);
    allClients = allClients.concat(response.data);

    if (response.pagination.page >= response.pagination.pages) {
      break;
    }

    page++;
  }

  console.log(`Exported ${allClients.length} clients`);

  // Convert to CSV
  const csv = [
    ['ID', 'Name', 'Email', 'Company', 'Created At'].join(','),
    ...allClients.map(c => [
      c.id,
      c.client_name,
      c.client_email,
      c.company_name || '',
      c.created_at
    ].join(','))
  ].join('\n');

  return csv;
}

// Usage
const csv = await exportAllClients();
console.log(csv);
```

## Error Handling

### Robust API Request with Retry

```javascript
async function robustApiRequest(method, endpoint, data = null, maxRetries = 3) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await uptillo[method](endpoint, data);

    } catch (error) {
      lastError = error;

      // Don't retry client errors (4xx)
      if (error.message.includes('API Error') && !error.message.includes('429')) {
        throw error;
      }

      // Retry with exponential backoff
      if (attempt < maxRetries) {
        const delay = Math.pow(2, attempt) * 1000;
        console.log(`Retry ${attempt}/${maxRetries} after ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError;
}

// Usage
try {
  const client = await robustApiRequest('post', '/clients', {
    client_name: 'ACME Corp',
    client_email: 'billing@acme.com'
  });
  console.log('Client created:', client.data.id);
} catch (error) {
  console.error('Failed after retries:', error.message);
}
```

## Next Steps

- [Authentication](authentication.md)
- [Webhooks](webhooks.md)
- [Error Handling](errors.md)
- [Rate Limiting](rate-limiting.md)
