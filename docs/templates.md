# Email Templates API

Customize reminder email templates with your own branding and messaging.

## Base Endpoint

```
/api/templates
```

## Template Object

```json
{
  "id": "tpl_abc123",
  "account_id": "acc_xyz789",
  "name": "3-Day Friendly Reminder",
  "subject": "Friendly reminder: Invoice {{invoice_number}} is overdue",
  "body_text": "Hi {{client_name}},\n\nJust a friendly reminder that invoice {{invoice_number}} for {{invoice_amount}} was due on {{due_date}}.\n\nBest regards,\n{{business_name}}",
  "body_html": "<p>Hi {{client_name}},</p><p>Just a friendly reminder...</p>",
  "template_type": "followup_3_days",
  "is_default": false,
  "is_active": true,
  "created_at": "2024-01-15T10:00:00.000Z",
  "updated_at": "2024-02-10T14:30:00.000Z"
}
```

### Attributes

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier for the template |
| `account_id` | string | Your account ID |
| `name` | string | Template name for internal reference |
| `subject` | string | Email subject line (supports variables) |
| `body_text` | string | Plain text email body (supports variables) |
| `body_html` | string (optional) | HTML email body (supports variables) |
| `template_type` | string | Template type: `followup_3_days`, `followup_7_days`, `followup_14_days`, `custom` |
| `is_default` | boolean | Whether this is the default template for its type |
| `is_active` | boolean | Whether this template is currently active |
| `created_at` | datetime | When the template was created |
| `updated_at` | datetime | When the template was last updated |

## Template Variables

Use these variables in your subject, body_text, and body_html:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{client_name}}` | Client's name | "ACME Corp" |
| `{{invoice_number}}` | Invoice number | "INV-2024-001" |
| `{{invoice_amount}}` | Formatted invoice amount | "€1,250.00" |
| `{{currency}}` | Currency code | "EUR" |
| `{{due_date}}` | Formatted due date | "March 15, 2024" |
| `{{business_name}}` | Your business name | "Your Company Inc" |
| `{{payment_link}}` | Payment link (if provided) | "https://pay.example.com/..." |
| `{{custom_note}}` | Custom note (if provided) | "Please reference PO #12345" |
| `{{smartlink}}` | Client response link | Auto-generated |
| `{{unsubscribe_link}}` | Unsubscribe link | Auto-generated |
| `{{business_address}}` | Your business address | Auto-generated |

## List All Templates

Retrieve all templates for your account.

```bash
GET /api/templates
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `template_type` | string | - | Filter by type (followup_3_days, followup_7_days, etc.) |
| `is_active` | boolean | - | Filter by active status |

### Example Request

```bash
curl https://uptillo.com/api/templates \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "data": [
    {
      "id": "tpl_abc123",
      "name": "3-Day Friendly Reminder",
      "subject": "Friendly reminder: Invoice {{invoice_number}} is overdue",
      "template_type": "followup_3_days",
      "is_default": true,
      "is_active": true,
      "created_at": "2024-01-15T10:00:00.000Z"
    },
    {
      "id": "tpl_def456",
      "name": "7-Day Important Notice",
      "subject": "Important: Invoice {{invoice_number}} is 7 days overdue",
      "template_type": "followup_7_days",
      "is_default": true,
      "is_active": true,
      "created_at": "2024-01-15T10:00:00.000Z"
    }
  ]
}
```

## Get a Single Template

Retrieve details of a specific template.

```bash
GET /api/templates/:template_id
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "tpl_abc123",
    "name": "3-Day Friendly Reminder",
    "subject": "Friendly reminder: Invoice {{invoice_number}} is overdue",
    "body_text": "Hi {{client_name}},\n\nJust a friendly reminder that invoice {{invoice_number}} for {{invoice_amount}} was due on {{due_date}}.\n\nYou can pay online here: {{payment_link}}\n\nBest regards,\n{{business_name}}",
    "body_html": "<p>Hi {{client_name}},</p><p>Just a friendly reminder that invoice <strong>{{invoice_number}}</strong> for <strong>{{invoice_amount}}</strong> was due on {{due_date}}.</p><p><a href=\"{{payment_link}}\">Pay Now</a></p>",
    "template_type": "followup_3_days",
    "is_default": true,
    "is_active": true,
    "created_at": "2024-01-15T10:00:00.000Z",
    "updated_at": "2024-02-10T14:30:00.000Z"
  }
}
```

## Create a Template

Create a new email template.

```bash
POST /api/templates
```

### Request Body

```json
{
  "name": "Custom 3-Day Reminder",
  "subject": "Hey {{client_name}}, quick reminder about invoice {{invoice_number}}",
  "body_text": "Hi {{client_name}},\n\nJust checking in about invoice {{invoice_number}} for {{invoice_amount}}. It was due on {{due_date}}.\n\nLet me know if you have any questions!\n\nThanks,\n{{business_name}}",
  "body_html": "<p>Hi {{client_name}},</p><p>Just checking in about invoice <strong>{{invoice_number}}</strong> for <strong>{{invoice_amount}}</strong>. It was due on {{due_date}}.</p><p>Let me know if you have any questions!</p><p>Thanks,<br>{{business_name}}</p>",
  "template_type": "followup_3_days",
  "is_active": true
}
```

### Required Fields

- `name` - Template name
- `subject` - Email subject line
- `body_text` - Plain text email body
- `template_type` - Template type (followup_3_days, followup_7_days, followup_14_days, custom)

### Example Request

```bash
curl https://uptillo.com/api/templates \
  -H "Authorization: Bearer pk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Custom 3-Day Reminder",
    "subject": "Reminder: Invoice {{invoice_number}}",
    "body_text": "Hi {{client_name}}, ...",
    "template_type": "followup_3_days"
  }'
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "tpl_new789",
    "name": "Custom 3-Day Reminder",
    "subject": "Reminder: Invoice {{invoice_number}}",
    "template_type": "followup_3_days",
    "is_active": true,
    "created_at": "2024-02-10T15:00:00.000Z"
  }
}
```

## Update a Template

Modify an existing template.

```bash
PATCH /api/templates/:template_id
```

### Request Body

```json
{
  "subject": "Updated: Reminder about invoice {{invoice_number}}",
  "body_text": "Updated text content...",
  "is_active": true
}
```

### Example Request

```bash
curl -X PATCH https://uptillo.com/api/templates/tpl_abc123 \
  -H "Authorization: Bearer pk_live_abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "Updated: Reminder about invoice {{invoice_number}}"
  }'
```

## Set as Default Template

Set a template as the default for its type.

```bash
POST /api/templates/:template_id/set-default
```

### Example Request

```bash
curl -X POST https://uptillo.com/api/templates/tpl_abc123/set-default \
  -H "Authorization: Bearer pk_live_abc123"
```

### Example Response

```json
{
  "success": true,
  "data": {
    "id": "tpl_abc123",
    "is_default": true
  },
  "message": "Template set as default for followup_3_days"
}
```

## Delete a Template

Delete a custom template.

**Note**: Default templates cannot be deleted, only deactivated.

```bash
DELETE /api/templates/:template_id
```

### Example Request

```bash
curl -X DELETE https://uptillo.com/api/templates/tpl_abc123 \
  -H "Authorization: Bearer pk_live_abc123"
```

## Preview Template

Preview a template with sample data.

```bash
POST /api/templates/:template_id/preview
```

### Request Body (Optional)

```json
{
  "client_name": "ACME Corp",
  "invoice_number": "INV-2024-001",
  "invoice_amount": 1250.00,
  "currency": "EUR",
  "due_date": "2024-03-15"
}
```

### Example Response

```json
{
  "success": true,
  "data": {
    "subject": "Friendly reminder: Invoice INV-2024-001 is overdue",
    "body_text": "Hi ACME Corp,\n\nJust a friendly reminder that invoice INV-2024-001 for €1,250.00 was due on March 15, 2024.\n\nBest regards,\nYour Company Inc",
    "body_html": "<p>Hi ACME Corp,</p><p>Just a friendly reminder that invoice <strong>INV-2024-001</strong> for <strong>€1,250.00</strong> was due on March 15, 2024.</p>"
  }
}
```

## Template Best Practices

### Subject Lines

✅ **Good Subject Lines:**
- "Friendly reminder: Invoice {{invoice_number}} is overdue"
- "Payment reminder for invoice {{invoice_number}}"
- "{{client_name}}, invoice {{invoice_number}} needs attention"

❌ **Avoid:**
- ALL CAPS: "URGENT!!! PAY NOW!!!"
- Excessive punctuation: "Invoice overdue!!!???"
- Deceptive subjects: "Re: Your payment" (when there's no prior conversation)

### Body Content

✅ **Good Practices:**
- Be polite and professional
- Include all invoice details (number, amount, due date)
- Provide clear payment instructions
- Offer to help with questions
- Include a payment link if available

❌ **Avoid:**
- Threatening language
- Excessive urgency
- Long paragraphs (keep it concise)
- Missing key information

### HTML Templates

When creating HTML templates:
- Use inline CSS (many email clients strip `<style>` tags)
- Keep HTML simple (avoid complex layouts)
- Always provide a plain text version
- Test across email clients

## Error Responses

### Template Not Found

```json
{
  "success": false,
  "error": {
    "code": "TEMPLATE_NOT_FOUND",
    "message": "No template found with the provided ID"
  }
}
```

HTTP Status: `404 Not Found`

### Invalid Template Variable

```json
{
  "success": false,
  "error": {
    "code": "INVALID_VARIABLE",
    "message": "Unknown template variable: {{invalid_var}}",
    "details": {
      "variable": "invalid_var",
      "valid_variables": ["client_name", "invoice_number", ...]
    }
  }
}
```

HTTP Status: `400 Bad Request`

## Code Examples

### JavaScript

```javascript
const axios = require('axios');

const apiKey = process.env.UPTILLO_API_KEY;
const baseURL = 'https://uptillo.com/api';

// Create a custom template
async function createTemplate() {
  const response = await axios.post(`${baseURL}/templates`, {
    name: 'Custom 3-Day Reminder',
    subject: 'Reminder: Invoice {{invoice_number}} is overdue',
    body_text: `Hi {{client_name}},

Just a friendly reminder that invoice {{invoice_number}} for {{invoice_amount}} was due on {{due_date}}.

You can pay online here: {{payment_link}}

Best regards,
{{business_name}}`,
    template_type: 'followup_3_days',
    is_active: true
  }, {
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    }
  });

  console.log('Template created:', response.data.data.id);
  return response.data.data;
}

// Preview template
async function previewTemplate(templateId) {
  const response = await axios.post(`${baseURL}/templates/${templateId}/preview`, {
    client_name: 'ACME Corp',
    invoice_number: 'INV-2024-001',
    invoice_amount: 1250.00,
    currency: 'EUR',
    due_date: '2024-03-15'
  }, {
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json'
    }
  });

  console.log('Preview:', response.data.data);
  return response.data.data;
}
```

### Python

```python
import requests
import os

api_key = os.getenv('UPTILLO_API_KEY')
base_url = 'https://uptillo.com/api'

# Create a custom template
def create_template():
    headers = {
        'Authorization': f'Bearer {api_key}',
        'Content-Type': 'application/json'
    }

    data = {
        'name': 'Custom 3-Day Reminder',
        'subject': 'Reminder: Invoice {{invoice_number}} is overdue',
        'body_text': '''Hi {{client_name}},

Just a friendly reminder that invoice {{invoice_number}} for {{invoice_amount}} was due on {{due_date}}.

Best regards,
{{business_name}}''',
        'template_type': 'followup_3_days',
        'is_active': True
    }

    response = requests.post(f'{base_url}/templates', json=data, headers=headers)

    if response.status_code == 200:
        template = response.json()['data']
        print(f"Template created: {template['id']}")
        return template
    else:
        print(f"Error: {response.json()}")
```

## Next Steps

- [Invoice Chases API](invoice-chases.md)
- [Webhooks](webhooks.md)
- [Code Examples](examples.md)
