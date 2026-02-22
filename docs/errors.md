# Error Handling

Comprehensive guide to handling errors in the Uptillo API.

## Error Response Format

All API errors follow a consistent JSON format:

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {
      "field": "field_name",
      "value": "invalid_value"
    }
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Always `false` for errors |
| `error.code` | string | Machine-readable error code |
| `error.message` | string | Human-readable error description |
| `error.details` | object (optional) | Additional error context |

## HTTP Status Codes

The Uptillo API uses standard HTTP status codes:

| Status Code | Meaning | Description |
|-------------|---------|-------------|
| `200` | OK | Request succeeded |
| `201` | Created | Resource created successfully |
| `400` | Bad Request | Invalid request parameters |
| `401` | Unauthorized | Missing or invalid API key |
| `403` | Forbidden | API key doesn't have required permissions |
| `404` | Not Found | Resource doesn't exist |
| `409` | Conflict | Resource already exists (duplicate) |
| `422` | Unprocessable Entity | Valid syntax but semantic errors |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Server error (rare) |
| `503` | Service Unavailable | Temporary server issue |

## Common Error Codes

### Authentication Errors

#### UNAUTHORIZED

Missing or invalid API key.

```json
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "API key is required"
  }
}
```

**HTTP Status:** `401`

**Solution:** Include valid API key in Authorization header:
```bash
Authorization: Bearer pk_live_your_key_here
```

#### INVALID_API_KEY

API key is malformed, revoked, or doesn't exist.

```json
{
  "success": false,
  "error": {
    "code": "INVALID_API_KEY",
    "message": "The provided API key is invalid or has been revoked"
  }
}
```

**HTTP Status:** `401`

**Solution:** Check your API key in the dashboard and generate a new one if needed.

#### ACCOUNT_SUSPENDED

Your account has been suspended.

```json
{
  "success": false,
  "error": {
    "code": "ACCOUNT_SUSPENDED",
    "message": "Your account has been suspended. Please contact support.",
    "details": {
      "suspended_at": "2024-02-10T15:00:00.000Z",
      "reason": "Payment failed"
    }
  }
}
```

**HTTP Status:** `403`

**Solution:** Contact support@uptillo.com to resolve.

### Validation Errors

#### VALIDATION_ERROR

Request contains invalid data.

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "client_email is required",
    "details": {
      "field": "client_email",
      "validation": "required"
    }
  }
}
```

**HTTP Status:** `400`

**Common Validation Issues:**
- Missing required fields
- Invalid email format
- Invalid date format (use YYYY-MM-DD or ISO 8601)
- Negative numbers for amounts
- Invalid currency code

#### INVALID_FORMAT

Field value has incorrect format.

```json
{
  "success": false,
  "error": {
    "code": "INVALID_FORMAT",
    "message": "Invalid email format",
    "details": {
      "field": "client_email",
      "value": "not-an-email"
    }
  }
}
```

**HTTP Status:** `400`

### Resource Errors

#### NOT_FOUND

Requested resource doesn't exist.

```json
{
  "success": false,
  "error": {
    "code": "CLIENT_NOT_FOUND",
    "message": "No client found with the provided ID"
  }
}
```

**HTTP Status:** `404`

**Variations:**
- `CLIENT_NOT_FOUND`
- `INVOICE_CHASE_NOT_FOUND`
- `TEMPLATE_NOT_FOUND`
- `API_KEY_NOT_FOUND`

#### DUPLICATE_RESOURCE

Resource with same identifier already exists.

```json
{
  "success": false,
  "error": {
    "code": "DUPLICATE_CLIENT",
    "message": "A client with this email already exists",
    "details": {
      "field": "client_email",
      "existing_client_id": "client_abc123"
    }
  }
}
```

**HTTP Status:** `409`

**Variations:**
- `DUPLICATE_CLIENT` - Client with same email exists
- `DUPLICATE_INVOICE` - Active chase for this invoice number exists

**Solution:** Use PATCH to update existing resource or check for duplicates first.

### Rate Limiting Errors

#### RATE_LIMIT_EXCEEDED

You've exceeded your API rate limit.

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Daily rate limit exceeded for this API key",
    "details": {
      "limit": 1000,
      "requests_today": 1000,
      "reset_at": "2024-02-11T00:00:00.000Z"
    }
  }
}
```

**HTTP Status:** `429`

**Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1707609600
Retry-After: 3600
```

**Solution:** Wait until reset time or upgrade your plan for higher limits.

### Business Logic Errors

#### CLIENT_OPTED_OUT

Client has unsubscribed from reminders.

```json
{
  "success": false,
  "error": {
    "code": "CLIENT_OPTED_OUT",
    "message": "This client has opted out of receiving reminders",
    "details": {
      "opted_out_at": "2024-02-05T10:30:00.000Z"
    }
  }
}
```

**HTTP Status:** `422`

**Solution:** Contact client directly to resolve.

#### INSUFFICIENT_CREDITS

Account has run out of email credits.

```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_CREDITS",
    "message": "Insufficient email credits. Please upgrade your plan.",
    "details": {
      "monthly_limit": 100,
      "emails_sent": 100,
      "reset_at": "2024-03-01T00:00:00.000Z"
    }
  }
}
```

**HTTP Status:** `422`

**Solution:** Upgrade plan or wait until monthly reset.

### Server Errors

#### INTERNAL_ERROR

Unexpected server error.

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred. Please try again.",
    "details": {
      "request_id": "req_abc123xyz"
    }
  }
}
```

**HTTP Status:** `500`

**Solution:** Retry the request. If issue persists, contact support with the request_id.

#### SERVICE_UNAVAILABLE

Temporary service disruption.

```json
{
  "success": false,
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "Service temporarily unavailable. Please try again shortly."
  }
}
```

**HTTP Status:** `503`

**Solution:** Retry with exponential backoff.

## Error Handling Best Practices

### 1. Always Check HTTP Status

```javascript
const response = await fetch('https://uptillo.com/api/clients', {
  headers: { 'Authorization': `Bearer ${apiKey}` }
});

if (!response.ok) {
  const error = await response.json();
  console.error('API Error:', error.error.code, error.error.message);
  throw new Error(error.error.message);
}

const data = await response.json();
```

### 2. Handle Specific Error Codes

```javascript
try {
  const client = await createClient(data);
} catch (error) {
  if (error.response) {
    const { code, message, details } = error.response.data.error;

    switch (code) {
      case 'DUPLICATE_CLIENT':
        console.log('Client exists, updating instead...');
        return updateClient(details.existing_client_id, data);

      case 'VALIDATION_ERROR':
        console.error('Validation failed:', details.field, message);
        throw new Error(`Invalid ${details.field}: ${message}`);

      case 'RATE_LIMIT_EXCEEDED':
        const retryAfter = details.reset_at;
        console.log(`Rate limited. Retry after ${retryAfter}`);
        await sleep(60000); // Wait 1 minute
        return createClient(data); // Retry

      default:
        throw error;
    }
  }
}
```

### 3. Implement Retry Logic

```javascript
async function apiRequest(url, options, maxRetries = 3) {
  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);

      if (response.ok) {
        return await response.json();
      }

      const error = await response.json();

      // Don't retry client errors (4xx)
      if (response.status >= 400 && response.status < 500) {
        throw error;
      }

      // Retry server errors (5xx)
      lastError = error;
      const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
      console.log(`Retry ${attempt}/${maxRetries} after ${delay}ms`);
      await sleep(delay);

    } catch (err) {
      lastError = err;
      if (attempt === maxRetries) throw err;
    }
  }

  throw lastError;
}
```

### 4. Log Errors for Debugging

```javascript
function logApiError(error, context) {
  const logEntry = {
    timestamp: new Date().toISOString(),
    context,
    error: {
      code: error.error?.code,
      message: error.error?.message,
      details: error.error?.details
    },
    request_id: error.error?.details?.request_id
  };

  console.error('API Error:', JSON.stringify(logEntry, null, 2));

  // Send to error tracking service
  if (process.env.SENTRY_DSN) {
    Sentry.captureException(error, { extra: logEntry });
  }
}
```

### 5. Provide User-Friendly Messages

```javascript
function getUserFriendlyMessage(errorCode) {
  const messages = {
    'DUPLICATE_CLIENT': 'This client already exists in your account.',
    'VALIDATION_ERROR': 'Please check your input and try again.',
    'RATE_LIMIT_EXCEEDED': 'You\'ve reached your daily limit. Please try again tomorrow.',
    'UNAUTHORIZED': 'Invalid API key. Please check your settings.',
    'CLIENT_OPTED_OUT': 'This client has unsubscribed from reminders.',
    'INSUFFICIENT_CREDITS': 'You\'ve used all your email credits. Upgrade to continue.'
  };

  return messages[errorCode] || 'An unexpected error occurred. Please try again.';
}
```

## Example: Complete Error Handling

```javascript
const axios = require('axios');

async function createClient(clientData) {
  try {
    const response = await axios.post('https://uptillo.com/api/clients', clientData, {
      headers: {
        'Authorization': `Bearer ${process.env.UPTILLO_API_KEY}`,
        'Content-Type': 'application/json'
      }
    });

    return response.data.data;

  } catch (error) {
    if (error.response) {
      const { code, message, details } = error.response.data.error;

      // Log for debugging
      console.error('API Error:', {
        code,
        message,
        details,
        status: error.response.status
      });

      // Handle specific errors
      switch (code) {
        case 'DUPLICATE_CLIENT':
          throw new Error(`Client with email ${details.value} already exists`);

        case 'VALIDATION_ERROR':
          throw new Error(`Invalid ${details.field}: ${message}`);

        case 'RATE_LIMIT_EXCEEDED':
          throw new Error(`Rate limit exceeded. Try again at ${details.reset_at}`);

        case 'UNAUTHORIZED':
          throw new Error('Invalid API key. Check your configuration.');

        default:
          throw new Error(`API Error: ${message}`);
      }
    } else if (error.request) {
      // Network error
      throw new Error('Network error. Please check your connection.');
    } else {
      // Client-side error
      throw error;
    }
  }
}
```

## Testing Error Scenarios

### Trigger Validation Error

```bash
curl -X POST https://uptillo.com/api/clients \
  -H "Authorization: Bearer pk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "client_name": "Test",
    "client_email": "invalid-email"
  }'
```

### Trigger Rate Limit Error

```bash
# Send 1001 requests quickly to exceed daily limit
for i in {1..1001}; do
  curl https://uptillo.com/api/clients \
    -H "Authorization: Bearer pk_live_abc123"
done
```

### Trigger Unauthorized Error

```bash
curl https://uptillo.com/api/clients \
  -H "Authorization: Bearer invalid_key"
```

## Support

If you encounter persistent errors or need help:

- **Email:** support@uptillo.com
- **Status Page:** https://status.uptillo.com
- **Documentation:** https://uptillo.com/docs

Include the `request_id` from error responses when contacting support.

## Next Steps

- [Rate Limiting](rate-limiting.md)
- [Authentication](authentication.md)
- [Code Examples](examples.md)
