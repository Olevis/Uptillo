# Webhooks

Receive real-time notifications when events occur in your Uptillo account.

## Overview

Webhooks allow your application to receive automatic notifications when certain events happen, such as:
- Client responds to an invoice reminder
- Email is delivered or bounces
- Invoice chase status changes
- Payment promised by client

## Webhook Setup

### Creating a Webhook Endpoint

1. Log in to your Uptillo dashboard
2. Navigate to **Settings > Webhooks**
3. Click **Add Webhook Endpoint**
4. Enter your endpoint URL (must be HTTPS)
5. Select events to subscribe to
6. Save and copy your webhook signing secret

### Endpoint Requirements

- **Must use HTTPS** (except localhost for testing)
- **Respond within 5 seconds** with HTTP 200
- **Handle duplicate events** (webhooks may be sent more than once)
- **Verify webhook signatures** for security

## Event Types

### Client Response Events

#### `invoice.response.submitted`

Triggered when a client responds to an invoice reminder via the smart link.

```json
{
  "event": "invoice.response.submitted",
  "timestamp": "2024-02-10T15:30:00.000Z",
  "data": {
    "id": "resp_abc123",
    "invoice_chase_id": "chase_def456",
    "account_id": "acc_xyz789",
    "response_type": "WILL_PAY",
    "scheduled_date": "2024-02-15T00:00:00.000Z",
    "message": "Will pay by end of week",
    "submitted_at": "2024-02-10T15:30:00.000Z",
    "invoice": {
      "invoice_number": "INV-2024-001",
      "invoice_amount": "1250.00",
      "currency": "EUR",
      "due_date": "2024-02-05T00:00:00.000Z"
    },
    "client": {
      "id": "client_ghi789",
      "client_name": "ACME Corp",
      "client_email": "billing@acme.com"
    }
  }
}
```

**Response Types:**
- `WILL_PAY` - Client promises to pay on a specific date
- `ALREADY_PAID` - Client claims they already paid
- `DISPUTE` - Client disputes the invoice
- `NEED_MORE_TIME` - Client needs more time to pay

### Email Events

#### `email.delivered`

Triggered when an email is successfully delivered.

```json
{
  "event": "email.delivered",
  "timestamp": "2024-02-10T09:05:00.000Z",
  "data": {
    "id": "log_abc123",
    "invoice_chase_id": "chase_def456",
    "recipient_email": "billing@acme.com",
    "subject": "Friendly reminder: Invoice INV-2024-001 is overdue",
    "sent_at": "2024-02-10T09:00:00.000Z",
    "delivered_at": "2024-02-10T09:05:00.000Z",
    "template_key": "followup_3_days"
  }
}
```

#### `email.opened`

Triggered when a recipient opens an email.

```json
{
  "event": "email.opened",
  "timestamp": "2024-02-10T10:15:00.000Z",
  "data": {
    "id": "log_abc123",
    "invoice_chase_id": "chase_def456",
    "recipient_email": "billing@acme.com",
    "opened_at": "2024-02-10T10:15:00.000Z",
    "user_agent": "Mozilla/5.0..."
  }
}
```

#### `email.clicked`

Triggered when a recipient clicks a link in the email.

```json
{
  "event": "email.clicked",
  "timestamp": "2024-02-10T10:16:00.000Z",
  "data": {
    "id": "log_abc123",
    "invoice_chase_id": "chase_def456",
    "recipient_email": "billing@acme.com",
    "clicked_at": "2024-02-10T10:16:00.000Z",
    "link_url": "https://uptillo.com/respond?token=..."
  }
}
```

#### `email.bounced`

Triggered when an email bounces.

```json
{
  "event": "email.bounced",
  "timestamp": "2024-02-10T09:10:00.000Z",
  "data": {
    "id": "log_abc123",
    "invoice_chase_id": "chase_def456",
    "recipient_email": "invalid@example.com",
    "bounce_type": "hard",
    "error_message": "Mailbox does not exist",
    "bounced_at": "2024-02-10T09:10:00.000Z"
  }
}
```

**Bounce Types:**
- `hard` - Permanent failure (invalid email address)
- `soft` - Temporary failure (mailbox full, server down)

### Invoice Chase Events

#### `invoice_chase.created`

Triggered when a new invoice chase is created.

```json
{
  "event": "invoice_chase.created",
  "timestamp": "2024-02-10T14:00:00.000Z",
  "data": {
    "id": "chase_abc123",
    "invoice_number": "INV-2024-001",
    "invoice_amount": "1250.00",
    "currency": "EUR",
    "due_date": "2024-02-05T00:00:00.000Z",
    "status": "ACTIVE",
    "client": {
      "id": "client_def456",
      "client_name": "ACME Corp",
      "client_email": "billing@acme.com"
    }
  }
}
```

#### `invoice_chase.status_changed`

Triggered when an invoice chase status changes.

```json
{
  "event": "invoice_chase.status_changed",
  "timestamp": "2024-02-10T16:00:00.000Z",
  "data": {
    "id": "chase_abc123",
    "invoice_number": "INV-2024-001",
    "old_status": "ACTIVE",
    "new_status": "PAID",
    "changed_at": "2024-02-10T16:00:00.000Z"
  }
}
```

### Opt-Out Events

#### `client.opted_out`

Triggered when a client unsubscribes from reminders.

```json
{
  "event": "client.opted_out",
  "timestamp": "2024-02-10T11:00:00.000Z",
  "data": {
    "client_email": "billing@acme.com",
    "opted_out_at": "2024-02-10T11:00:00.000Z",
    "reason": "Already paid all invoices",
    "affected_chases": [
      {
        "id": "chase_abc123",
        "invoice_number": "INV-2024-001"
      }
    ]
  }
}
```

## Webhook Signature Verification

All webhook requests include a signature in the `X-Uptillo-Signature` header. Verify this signature to ensure the webhook came from Uptillo.

### Signature Format

```
X-Uptillo-Signature: t=1707571200,v1=5f9b2c1e8d7a6b4c3e2f1a0b9c8d7e6f...
```

### Verification Algorithm

1. Extract timestamp (`t`) and signature (`v1`) from header
2. Construct signed payload: `{timestamp}.{raw_body}`
3. Compute HMAC SHA256 using your webhook secret
4. Compare computed signature with provided signature

### Example Code

#### Node.js

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const parts = signature.split(',');
  const timestamp = parts.find(p => p.startsWith('t=')).split('=')[1];
  const receivedSig = parts.find(p => p.startsWith('v1=')).split('=')[1];

  // Check timestamp (prevent replay attacks)
  const now = Math.floor(Date.now() / 1000);
  if (Math.abs(now - parseInt(timestamp)) > 300) { // 5 minutes
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

// Express.js example
app.post('/webhooks/uptillo', express.raw({ type: 'application/json' }), (req, res) => {
  const signature = req.headers['x-uptillo-signature'];
  const webhookSecret = process.env.UPTILLO_WEBHOOK_SECRET;

  try {
    verifyWebhookSignature(req.body.toString(), signature, webhookSecret);

    const event = JSON.parse(req.body);

    // Handle event
    switch (event.event) {
      case 'invoice.response.submitted':
        handleClientResponse(event.data);
        break;
      case 'email.delivered':
        handleEmailDelivered(event.data);
        break;
      // ... other events
    }

    res.status(200).json({ received: true });
  } catch (error) {
    console.error('Webhook verification failed:', error.message);
    res.status(400).json({ error: 'Invalid signature' });
  }
});
```

#### Python (Flask)

```python
import hmac
import hashlib
import time

def verify_webhook_signature(payload, signature, secret):
    parts = dict(part.split('=') for part in signature.split(','))
    timestamp = parts['t']
    received_sig = parts['v1']

    # Check timestamp (prevent replay attacks)
    now = int(time.time())
    if abs(now - int(timestamp)) > 300:  # 5 minutes
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

# Flask example
@app.route('/webhooks/uptillo', methods=['POST'])
def webhook():
    signature = request.headers.get('X-Uptillo-Signature')
    webhook_secret = os.getenv('UPTILLO_WEBHOOK_SECRET')

    try:
        verify_webhook_signature(request.data.decode(), signature, webhook_secret)

        event = request.json

        # Handle event
        if event['event'] == 'invoice.response.submitted':
            handle_client_response(event['data'])
        elif event['event'] == 'email.delivered':
            handle_email_delivered(event['data'])
        # ... other events

        return {'received': True}, 200
    except ValueError as e:
        print(f'Webhook verification failed: {e}')
        return {'error': 'Invalid signature'}, 400
```

## Handling Webhooks

### Best Practices

✅ **DO:**
- Respond quickly with HTTP 200 (within 5 seconds)
- Process webhooks asynchronously (use a queue)
- Store raw webhook data for debugging
- Handle duplicate events (use event ID for idempotency)
- Verify webhook signatures
- Log all webhook events
- Implement retry logic for failed processing

❌ **DON'T:**
- Perform long-running tasks synchronously
- Return errors for successfully received webhooks
- Trust webhooks without signature verification
- Assume events arrive in order
- Ignore duplicate events

### Example: Processing Asynchronously

```javascript
// Express.js with job queue
app.post('/webhooks/uptillo', express.json(), async (req, res) => {
  const signature = req.headers['x-uptillo-signature'];

  try {
    // Verify signature
    verifyWebhookSignature(JSON.stringify(req.body), signature, process.env.UPTILLO_WEBHOOK_SECRET);

    // Respond immediately
    res.status(200).json({ received: true });

    // Process asynchronously
    await jobQueue.add('processWebhook', {
      event: req.body,
      receivedAt: new Date()
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

## Retry Logic

If your endpoint returns an error or times out, Uptillo will retry the webhook:

- **Retry Schedule**: 1 hour, 6 hours, 24 hours
- **Max Retries**: 3 attempts
- **Timeout**: 5 seconds per attempt

After 3 failed attempts, the webhook is marked as failed and you'll receive an email notification.

## Testing Webhooks

### Localhost Testing

Use a tool like [ngrok](https://ngrok.com) to expose your local server:

```bash
ngrok http 3000
```

Then use the ngrok URL as your webhook endpoint:
```
https://abc123.ngrok.io/webhooks/uptillo
```

### Manual Testing

Send a test webhook from the dashboard:

1. Go to **Settings > Webhooks**
2. Click on your webhook endpoint
3. Click **Send Test Event**
4. Select event type
5. View the response

### Example Test Event

```bash
curl -X POST https://your-app.com/webhooks/uptillo \
  -H "Content-Type: application/json" \
  -H "X-Uptillo-Signature: t=1707571200,v1=..." \
  -d '{
    "event": "invoice.response.submitted",
    "timestamp": "2024-02-10T15:30:00.000Z",
    "data": {
      "id": "resp_test123",
      "invoice_chase_id": "chase_test456",
      "response_type": "WILL_PAY",
      "scheduled_date": "2024-02-15T00:00:00.000Z",
      "message": "Test response"
    }
  }'
```

## Webhook Logs

View webhook delivery logs in your dashboard:

1. Go to **Settings > Webhooks**
2. Click on your webhook endpoint
3. View **Recent Deliveries**

Each log entry shows:
- Event type
- Timestamp
- HTTP response code
- Response time
- Response body
- Retry attempts

## Error Responses

### Invalid Signature

If signature verification fails, return:

```json
{
  "error": "Invalid signature"
}
```

HTTP Status: `400 Bad Request`

### Processing Error

If webhook processing fails, return:

```json
{
  "error": "Failed to process webhook",
  "details": "Database connection error"
}
```

HTTP Status: `500 Internal Server Error`

Uptillo will retry the webhook automatically.

## Security Considerations

1. **Always verify signatures** - Never trust webhooks without verification
2. **Use HTTPS** - Webhooks are only sent to HTTPS endpoints (except localhost)
3. **Check timestamps** - Reject webhooks with old timestamps (> 5 minutes)
4. **Rate limiting** - Implement rate limiting on webhook endpoints
5. **Idempotency** - Use event IDs to prevent duplicate processing
6. **Secret rotation** - Rotate webhook secrets periodically

## Next Steps

- [Invoice Chases API](invoice-chases.md)
- [Client Response Handling](examples.md#handling-client-responses)
- [Error Handling](errors.md)
