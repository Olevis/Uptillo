# Authentication

All Uptillo API requests require authentication using API keys.

## API Keys

API keys are managed from your Uptillo dashboard under **Settings > API Keys**.

### Key Format

```
pk_live_1a2b3c4d5e6f7g8h9i0j  # Production key
pk_test_9i8h7g6f5e4d3c2b1a0j  # Test key (sandbox)
```

### Creating an API Key

1. Log in to your Uptillo dashboard
2. Navigate to **Settings > API Keys**
3. Click **Create API Key**
4. Give your key a descriptive name (e.g., "Production Server", "Mobile App")
5. Copy the key immediately - it will only be shown once

### Authentication Header

Include your API key in the `Authorization` header with the `Bearer` scheme:

```bash
Authorization: Bearer pk_live_1a2b3c4d5e6f7g8h9i0j
```

## Example Requests

### cURL

```bash
curl https://uptillo.com/api/clients \
  -H "Authorization: Bearer pk_live_1a2b3c4d5e6f7g8h9i0j"
```

### JavaScript (fetch)

```javascript
const response = await fetch('https://uptillo.com/api/clients', {
  headers: {
    'Authorization': 'Bearer pk_live_1a2b3c4d5e6f7g8h9i0j',
    'Content-Type': 'application/json'
  }
});

const data = await response.json();
```

### Python (requests)

```python
import requests

headers = {
    'Authorization': 'Bearer pk_live_1a2b3c4d5e6f7g8h9i0j',
    'Content-Type': 'application/json'
}

response = requests.get('https://uptillo.com/api/clients', headers=headers)
data = response.json()
```

### PHP

```php
<?php
$api_key = 'pk_live_1a2b3c4d5e6f7g8h9i0j';

$ch = curl_init('https://uptillo.com/api/clients');
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'Authorization: Bearer ' . $api_key,
    'Content-Type: application/json'
]);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

$response = curl_exec($ch);
$data = json_decode($response, true);
curl_close($ch);
?>
```

## API Key Management

### View Active Keys

```bash
GET /api/keys
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "key_abc123",
      "name": "Production Server",
      "key_prefix": "pk_live_1a2b",
      "last_used_at": "2024-02-10T14:30:00.000Z",
      "requests_today": 234,
      "daily_limit": 1000,
      "is_active": true,
      "created_at": "2024-01-15T10:00:00.000Z"
    }
  ]
}
```

### Revoke an API Key

```bash
DELETE /api/keys/:key_id
```

**Response:**

```json
{
  "success": true,
  "message": "API key revoked successfully"
}
```

## Security Best Practices

### ✅ DO:

- **Store keys securely** - Use environment variables, never commit to version control
- **Use separate keys** - Different keys for development, staging, and production
- **Rotate regularly** - Create new keys every 90 days
- **Monitor usage** - Check API key activity in dashboard
- **Revoke compromised keys** - Immediately revoke any exposed keys

### ❌ DON'T:

- **Commit to Git** - Never include API keys in source code
- **Share keys** - Each integration should have its own key
- **Use in client-side code** - API keys should only be used server-side
- **Hardcode keys** - Always use environment variables

## Environment Variables

### Node.js

```javascript
// .env file
UPTILLO_API_KEY=pk_live_1a2b3c4d5e6f7g8h9i0j

// server.js
const apiKey = process.env.UPTILLO_API_KEY;
```

### Python

```python
# .env file
UPTILLO_API_KEY=pk_live_1a2b3c4d5e6f7g8h9i0j

# app.py
import os
api_key = os.getenv('UPTILLO_API_KEY')
```

### PHP

```php
// .env file
UPTILLO_API_KEY=pk_live_1a2b3c4d5e6f7g8h9i0j

// config.php
$api_key = getenv('UPTILLO_API_KEY');
```

## Error Responses

### Missing API Key

```json
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "API key is required"
  }
}
```

HTTP Status: `401 Unauthorized`

### Invalid API Key

```json
{
  "success": false,
  "error": {
    "code": "INVALID_API_KEY",
    "message": "The provided API key is invalid or has been revoked"
  }
}
```

HTTP Status: `401 Unauthorized`

### Rate Limit Exceeded

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Daily rate limit exceeded for this API key",
    "details": {
      "limit": 1000,
      "reset_at": "2024-02-11T00:00:00.000Z"
    }
  }
}
```

HTTP Status: `429 Too Many Requests`

## Testing

### Test Mode

Use test API keys (prefix `pk_test_`) to test integrations without affecting production data.

**Test Key Features:**
- Separate database from production
- No emails are actually sent
- Full API functionality available
- Rate limits still apply

### Verify API Key

Check if your API key is valid:

```bash
GET /api/auth/verify
```

**Response:**

```json
{
  "success": true,
  "data": {
    "account_id": "acc_abc123",
    "business_name": "ACME Inc",
    "plan": "PRO",
    "api_key_id": "key_abc123",
    "daily_limit": 1000,
    "requests_today": 145
  }
}
```

## Next Steps

- [Clients API Documentation](clients.md)
- [Invoice Chases API Documentation](invoice-chases.md)
- [Rate Limiting](rate-limiting.md)
