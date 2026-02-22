# Uptillo API Documentation

Complete REST API documentation for the Uptillo invoice reminder automation platform.

## Overview

Uptillo provides a powerful API for automating invoice reminder workflows. Send automated follow-ups, track client responses, and manage payment reminders programmatically.

**Base URL**: `https://uptillo.com/api`

**API Version**: v1

**Authentication**: API Key (Bearer Token)

## Quick Start

```bash
# Get your API key from dashboard settings
curl https://uptillo.com/api/clients \
  -H "Authorization: Bearer your_api_key_here"
```

## Table of Contents

- [Authentication](docs/authentication.md)
- [Clients Management](docs/clients.md)
- [Invoice Chases](docs/invoice-chases.md)
- [Email Templates](docs/templates.md)
- [Webhooks](docs/webhooks.md)
- [Rate Limiting](docs/rate-limiting.md)
- [Error Handling](docs/errors.md)
- [Code Examples](docs/examples.md)

## Core Concepts

### Clients

Clients represent the recipients of your invoice reminders. Each client belongs to your account and includes contact information.

```javascript
{
  "id": "cljk1x2y3000008l7abc123",
  "client_name": "ACME Corp",
  "client_email": "billing@acme.com",
  "company_name": "ACME Corporation",
  "notes": "Net 30 payment terms"
}
```

### Invoice Chases

Invoice chases are automated reminder workflows for specific invoices. Set the due date and amount, and Uptillo handles the rest.

```javascript
{
  "id": "cljk1x2y3000008l7def456",
  "invoice_number": "INV-2024-001",
  "invoice_amount": "1250.00",
  "currency": "EUR",
  "due_date": "2024-03-15T00:00:00.000Z",
  "status": "ACTIVE",
  "emails_sent_count": 2
}
```

### Email Templates

Customize reminder emails with your branding and messaging. Templates support dynamic variables like invoice number, amount, and due date.

## Features

✅ **Automated Reminders** - Schedule follow-ups 3, 7, and 14 days after due date
✅ **Client Management** - Organize and track all invoice recipients
✅ **Smart Links** - Track client responses ("Will pay on...", "Already paid", etc.)
✅ **CAN-SPAM Compliant** - Built-in unsubscribe and compliance features
✅ **Webhook Events** - Real-time notifications for payment responses
✅ **Rate Limiting** - Fair usage with 1000 requests/day per API key

## Authentication

All API requests require authentication using an API key. Include your key in the `Authorization` header:

```bash
Authorization: Bearer pk_live_abc123...
```

Get your API key from **Settings > API Keys** in the Uptillo dashboard.

[Full Authentication Documentation →](docs/authentication.md)

## Basic Example

### Create a Client and Start an Invoice Chase

```javascript
// 1. Create a client
const client = await fetch('https://uptillo.com/api/clients', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer pk_live_abc123',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    client_name: 'ACME Corp',
    client_email: 'billing@acme.com',
    company_name: 'ACME Corporation'
  })
});

const { data: clientData } = await client.json();

// 2. Create an invoice chase
const chase = await fetch('https://uptillo.com/api/invoice-chases', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer pk_live_abc123',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    client_id: clientData.id,
    invoice_number: 'INV-2024-001',
    invoice_amount: 1250.00,
    currency: 'EUR',
    due_date: '2024-03-15',
    payment_link: 'https://pay.example.com/inv-001'
  })
});

const { data: chaseData } = await chase.json();
console.log(`Invoice chase created: ${chaseData.id}`);
```

## Response Format

All API responses follow a consistent format:

### Success Response

```json
{
  "success": true,
  "data": {
    "id": "cljk1x2y3000008l7abc123",
    "client_name": "ACME Corp"
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "client_email is required",
    "details": {
      "field": "client_email"
    }
  }
}
```

## Rate Limits

- **Default**: 1000 requests per day per API key
- **Burst**: 60 requests per minute
- **Headers**: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`

[Full Rate Limiting Documentation →](docs/rate-limiting.md)

## Webhooks

Receive real-time notifications when clients respond to invoice reminders:

```json
{
  "event": "invoice.response.submitted",
  "data": {
    "invoice_chase_id": "cljk1x2y3000008l7def456",
    "response_type": "WILL_PAY",
    "scheduled_date": "2024-03-20T00:00:00.000Z",
    "message": "Will pay by end of week"
  }
}
```

[Full Webhook Documentation →](docs/webhooks.md)

## Support

- **Email**: support@uptillo.com
- **Documentation**: https://uptillo.com/docs
- **Status**: https://status.uptillo.com

## SDKs

### Official SDKs (Coming Soon)

- JavaScript/TypeScript
- Python
- PHP
- Ruby

For now, use the REST API directly with your preferred HTTP client.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and updates.

## License

This API documentation is provided under the [MIT License](LICENSE).

---

**Built with ❤️ by the Uptillo team**
