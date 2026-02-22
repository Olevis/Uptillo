# Rate Limiting

Understand and work within Uptillo's API rate limits.

## Overview

Rate limiting ensures fair usage and protects the API from abuse. All API endpoints are subject to rate limits based on your plan.

## Rate Limit Tiers

| Plan | Daily Limit | Burst Limit | Email Credits/Month |
|------|-------------|-------------|---------------------|
| **Trial** | 100 requests/day | 10 req/min | 30 emails |
| **Starter** | 1,000 requests/day | 60 req/min | 100 emails |
| **Pro** | 10,000 requests/day | 120 req/min | 500 emails |
| **Enterprise** | Custom | Custom | Custom |

## Rate Limit Headers

Every API response includes rate limit information in the headers:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1707609600
```

### Header Descriptions

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Total requests allowed per day |
| `X-RateLimit-Remaining` | Requests remaining in current period |
| `X-RateLimit-Reset` | Unix timestamp when limit resets (midnight UTC) |

## How Rate Limiting Works

### Daily Limit

Resets at midnight UTC every day.

```javascript
// Check rate limit headers
const response = await fetch('https://uptillo.com/api/clients', {
  headers: { 'Authorization': `Bearer ${apiKey}` }
});

const limit = response.headers.get('X-RateLimit-Limit');
const remaining = response.headers.get('X-RateLimit-Remaining');
const reset = response.headers.get('X-RateLimit-Reset');

console.log(`Remaining: ${remaining}/${limit}`);
console.log(`Resets at: ${new Date(reset * 1000).toISOString()}`);
```

### Burst Limit

Prevents rapid-fire requests within a short time window.

- **Window:** 1 minute
- **Action:** If exceeded, requests are delayed or rejected

## Rate Limit Exceeded Response

When you exceed your rate limit, you'll receive a `429 Too Many Requests` response:

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

**Headers:**
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1707609600
Retry-After: 3600
```

## Best Practices

### 1. Monitor Rate Limit Headers

Always check remaining requests before making critical calls:

```javascript
async function makeApiRequest(url, options) {
  const response = await fetch(url, options);

  const remaining = parseInt(response.headers.get('X-RateLimit-Remaining'));

  if (remaining < 10) {
    console.warn('Rate limit almost reached!', remaining);
    // Alert or pause requests
  }

  return response.json();
}
```

### 2. Implement Exponential Backoff

When rate limited, retry with increasing delays:

```javascript
async function apiRequestWithBackoff(url, options, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);

      if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After');
        const delay = retryAfter ? parseInt(retryAfter) * 1000 : Math.pow(2, attempt) * 1000;

        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await sleep(delay);
        continue;
      }

      return await response.json();

    } catch (error) {
      if (attempt === maxRetries) throw error;
    }
  }
}

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

### 3. Batch Requests

Instead of making multiple individual requests, batch related operations:

❌ **Bad: Multiple Individual Requests**
```javascript
// Makes 100 API calls
for (const client of clients) {
  await createClient(client);
}
```

✅ **Good: Bulk Create (if available)**
```javascript
// Makes 1 API call
await bulkCreateClients(clients);
```

### 4. Cache Responses

Cache data that doesn't change frequently:

```javascript
const cache = new Map();
const CACHE_TTL = 60 * 60 * 1000; // 1 hour

async function getCachedClients() {
  const cacheKey = 'all_clients';
  const cached = cache.get(cacheKey);

  if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
    return cached.data;
  }

  const response = await fetch('https://uptillo.com/api/clients', {
    headers: { 'Authorization': `Bearer ${apiKey}` }
  });

  const data = await response.json();

  cache.set(cacheKey, {
    data: data.data,
    timestamp: Date.now()
  });

  return data.data;
}
```

### 5. Use Webhooks Instead of Polling

Instead of repeatedly checking for updates, use webhooks:

❌ **Bad: Polling**
```javascript
// Check for new responses every minute
setInterval(async () => {
  const responses = await fetch('https://uptillo.com/api/invoice-chases/chase_123/emails');
  // Uses 1440 requests per day!
}, 60000);
```

✅ **Good: Webhooks**
```javascript
// Receive real-time notifications via webhook
app.post('/webhooks/uptillo', (req, res) => {
  if (req.body.event === 'invoice.response.submitted') {
    handleNewResponse(req.body.data);
  }
  res.status(200).json({ received: true });
});
// Uses 0 API requests!
```

### 6. Prioritize Critical Requests

If you're close to the limit, prioritize essential operations:

```javascript
async function smartApiRequest(url, options, priority = 'normal') {
  const response = await fetch('https://uptillo.com/api/auth/verify', {
    headers: { 'Authorization': `Bearer ${apiKey}` }
  });

  const remaining = parseInt(response.headers.get('X-RateLimit-Remaining'));

  // Reserve last 10% for critical operations
  const threshold = parseInt(response.headers.get('X-RateLimit-Limit')) * 0.1;

  if (remaining < threshold && priority !== 'critical') {
    throw new Error('Rate limit buffer - critical operations only');
  }

  return fetch(url, options);
}

// Usage
await smartApiRequest('/api/invoice-chases', { method: 'POST' }, 'critical');
await smartApiRequest('/api/clients', { method: 'GET' }, 'normal');
```

## Rate Limit Per API Key

Each API key has its own rate limit. Use multiple keys for different services:

```javascript
// Separate keys for different purposes
const DASHBOARD_API_KEY = process.env.UPTILLO_DASHBOARD_KEY;
const CRON_API_KEY = process.env.UPTILLO_CRON_KEY;
const WEBHOOK_PROCESSOR_KEY = process.env.UPTILLO_WEBHOOK_KEY;

// Dashboard queries
const clients = await fetch('https://uptillo.com/api/clients', {
  headers: { 'Authorization': `Bearer ${DASHBOARD_API_KEY}` }
});

// Background jobs
const chase = await fetch('https://uptillo.com/api/invoice-chases', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${CRON_API_KEY}` }
});
```

## Upgrading for Higher Limits

Need more requests? Upgrade your plan:

1. Log in to your Uptillo dashboard
2. Go to **Settings > Billing**
3. Click **Upgrade Plan**
4. Choose **Pro** (10,000 requests/day) or **Enterprise** (custom limits)

### Enterprise Custom Limits

For high-volume integrations, contact sales for custom rate limits:
- **Email:** sales@uptillo.com
- **Custom Limits:** 100,000+ requests/day
- **Dedicated Support:** Priority assistance
- **SLA:** 99.9% uptime guarantee

## Monitoring Rate Limit Usage

### Dashboard

View rate limit usage in real-time:

1. Go to **Settings > API Keys**
2. Select your API key
3. View **Daily Usage Graph**

### Programmatic Monitoring

```javascript
async function checkRateLimitStatus(apiKey) {
  const response = await fetch('https://uptillo.com/api/auth/verify', {
    headers: { 'Authorization': `Bearer ${apiKey}` }
  });

  const data = await response.json();

  return {
    limit: data.data.daily_limit,
    used: data.data.requests_today,
    remaining: data.data.daily_limit - data.data.requests_today,
    resetAt: data.data.reset_at,
    percentage: (data.data.requests_today / data.data.daily_limit) * 100
  };
}

// Usage
const status = await checkRateLimitStatus(process.env.UPTILLO_API_KEY);

if (status.percentage > 80) {
  console.warn('Warning: 80% of rate limit used!');
  // Send alert notification
}
```

### Alerts

Set up alerts when approaching rate limit:

```javascript
const ALERT_THRESHOLDS = [50, 75, 90, 95];

async function monitorRateLimit() {
  const status = await checkRateLimitStatus(apiKey);

  for (const threshold of ALERT_THRESHOLDS) {
    if (status.percentage >= threshold && status.percentage < threshold + 5) {
      await sendAlert({
        level: threshold >= 90 ? 'critical' : 'warning',
        message: `Rate limit ${threshold}% used (${status.remaining} remaining)`,
        resetAt: status.resetAt
      });
    }
  }
}

// Run every hour
setInterval(monitorRateLimit, 60 * 60 * 1000);
```

## Email Credits vs API Rate Limit

**API Rate Limit:** Number of API requests you can make per day

**Email Credits:** Number of emails you can send per month

These are **separate limits**:

```javascript
// This counts towards API rate limit
const response = await fetch('https://uptillo.com/api/invoice-chases', {
  method: 'POST',
  body: JSON.stringify({ /* ... */ })
});
// API requests used: 1
// Email credits used: 0 (no email sent yet)

// When Uptillo sends the first reminder (automatically)
// API requests used: 1 (no additional API call)
// Email credits used: 1
```

## FAQs

**Q: Does the rate limit reset exactly at midnight?**
A: Yes, at midnight UTC. Check `X-RateLimit-Reset` header for exact timestamp.

**Q: What happens if I exceed the burst limit?**
A: Requests are delayed by a few seconds or rejected with 429 status.

**Q: Do webhook deliveries count towards rate limit?**
A: No, incoming webhooks don't count. Only API requests you make count.

**Q: Can I request a temporary rate limit increase?**
A: Yes, contact support@uptillo.com with your use case.

**Q: Do failed requests count towards the limit?**
A: Yes, all requests (successful or failed) count towards the limit.

## Code Example: Complete Rate Limit Handler

```javascript
class UptilloClient {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseURL = 'https://uptillo.com/api';
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const defaultOptions = {
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    };

    let retries = 3;

    while (retries > 0) {
      try {
        const response = await fetch(url, defaultOptions);

        // Log rate limit status
        this.logRateLimit(response.headers);

        if (response.status === 429) {
          const retryAfter = parseInt(response.headers.get('Retry-After') || '60');
          console.log(`Rate limited. Retrying after ${retryAfter}s...`);
          await this.sleep(retryAfter * 1000);
          retries--;
          continue;
        }

        if (!response.ok) {
          const error = await response.json();
          throw new Error(error.error.message);
        }

        return await response.json();

      } catch (error) {
        if (retries === 1) throw error;
        retries--;
        await this.sleep(Math.pow(2, 4 - retries) * 1000);
      }
    }
  }

  logRateLimit(headers) {
    const limit = headers.get('X-RateLimit-Limit');
    const remaining = headers.get('X-RateLimit-Remaining');
    const reset = headers.get('X-RateLimit-Reset');

    const percentage = ((limit - remaining) / limit) * 100;

    if (percentage > 90) {
      console.warn(`⚠️  Rate limit: ${remaining}/${limit} remaining`);
    }
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
const client = new UptilloClient(process.env.UPTILLO_API_KEY);

try {
  const clients = await client.request('/clients');
  console.log('Clients:', clients.data);
} catch (error) {
  console.error('API Error:', error.message);
}
```

## Next Steps

- [Error Handling](errors.md)
- [Authentication](authentication.md)
- [Webhooks](webhooks.md)
