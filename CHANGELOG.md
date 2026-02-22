# Changelog

All notable changes to the Uptillo API will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2024-02-10

### Added

- **Clients API**: Create, read, update, and delete client contacts
- **Invoice Chases API**: Automated invoice reminder workflows
- **Email Templates API**: Customize reminder email templates
- **Webhooks**: Real-time event notifications
  - `invoice.response.submitted` - Client responds to reminder
  - `email.delivered` - Email successfully delivered
  - `email.opened` - Client opens email
  - `email.clicked` - Client clicks link in email
  - `email.bounced` - Email bounces
  - `invoice_chase.created` - New invoice chase created
  - `invoice_chase.status_changed` - Invoice status changes
  - `client.opted_out` - Client unsubscribes
- **API Key Management**: Create, list, and revoke API keys
- **Rate Limiting**: 1,000 requests/day (Starter), 10,000/day (Pro)
- **CAN-SPAM Compliance**: Automatic unsubscribe links and business address inclusion
- **Client Response System**: Smart links for clients to respond with payment status

### Features

- Automatic reminder schedule: 3, 7, and 14 days after due date
- Multi-currency support (EUR, USD, GBP, and more)
- Email tracking (delivery, opens, clicks)
- Opt-out management
- Custom email templates with variable substitution
- Webhook signature verification for security
- Comprehensive error handling with detailed error codes

### Security

- API key authentication with Bearer tokens
- Webhook signature verification (HMAC SHA256)
- Rate limiting to prevent abuse
- IP address and user agent hashing for privacy

### Documentation

- Complete REST API documentation
- Code examples in JavaScript, Python, and PHP
- Webhook integration guide
- Error handling guide
- Rate limiting guide

---

## Upcoming Features

### [1.1.0] - Planned Q2 2024

- **Bulk Operations API**
  - `POST /api/clients/bulk` - Create multiple clients
  - `POST /api/invoice-chases/bulk` - Create multiple chases
  - `PATCH /api/invoice-chases/bulk` - Update multiple chases
- **Enhanced Filtering**
  - Date range filters for invoice chases
  - Advanced search with multiple criteria
  - Custom field sorting
- **Analytics API**
  - `GET /api/analytics/overview` - Account-wide statistics
  - `GET /api/analytics/clients/:id` - Per-client analytics
  - `GET /api/analytics/emails` - Email performance metrics
- **Scheduled Reminders**
  - Custom reminder schedules (beyond 3, 7, 14 days)
  - One-time scheduled emails
  - Recurring reminder patterns

### [1.2.0] - Planned Q3 2024

- **Multi-Language Support**
  - Template translations
  - Automatic language detection
  - Custom language overrides
- **Advanced Templates**
  - Conditional content blocks
  - A/B testing support
  - Dynamic content insertion
- **Team Collaboration**
  - Team member API keys
  - Role-based permissions
  - Activity logs
- **Integrations**
  - Native QuickBooks integration
  - Xero accounting integration
  - Zapier actions and triggers

### [2.0.0] - Planned Q4 2024

- **GraphQL API** (alongside REST)
- **Real-time API** (WebSocket support)
- **Enhanced Webhooks**
  - Webhook retry configuration
  - Custom retry schedules
  - Webhook filtering by event type
- **Advanced Security**
  - OAuth 2.0 support
  - IP whitelisting
  - API key expiration policies

---

## Previous Versions

### [0.9.0] - 2024-01-15 (Beta)

- Initial beta release
- Basic client and invoice chase management
- Simple email templates
- Limited webhook support

---

## Migration Guides

### Migrating to v1.0.0 from v0.9.0

**Breaking Changes:**
- None (v1.0.0 is backward compatible with v0.9.0)

**New Features:**
- Webhooks now include signature verification - update your webhook handlers
- New template variables available: `{{unsubscribe_link}}`, `{{business_address}}`
- Rate limiting headers now included in all responses

**Recommendations:**
1. Update webhook handlers to verify signatures (see [Webhooks documentation](docs/webhooks.md))
2. Review and update email templates to include new variables
3. Monitor rate limit headers in API responses

---

## Support

- **Email:** support@uptillo.com
- **Documentation:** https://uptillo.com/docs
- **Status:** https://status.uptillo.com
- **GitHub:** https://github.com/uptillo/api-docs (this repository)

---

## Version Support

| Version | Status | Supported Until |
|---------|--------|----------------|
| 1.0.x | ✅ Current | Ongoing |
| 0.9.x | ⚠️ Deprecated | 2024-06-30 |

We recommend all users upgrade to the latest version to receive the latest features and security updates.
