# Neoleap API Reference

## Base URLs

- **Production**: `https://api.neoleap.com/v1`
- **Sandbox**: `https://sandbox-api.neoleap.com/v1`

## Authentication

All API requests require authentication using API keys.

### Headers

```
Authorization: Bearer {API_KEY}
Content-Type: application/json
X-Neoleap-Signature: {HMAC_SHA256_SIGNATURE}
```

### Request Signing

Generate HMAC signature for request validation:

```javascript
const crypto = require('crypto');

const payload = JSON.stringify(requestBody);
const timestamp = Date.now();
const signaturePayload = `${timestamp}.${payload}`;
const signature = crypto
  .createHmac('sha256', apiSecret)
  .update(signaturePayload)
  .digest('hex');
```

## Endpoints

### Channels API

#### POST /channels

Create a new payment channel.

**Request Body:**
```json
{
  "type": "card|wallet|bank",
  "name": "string",
  "config": {
    "enabled3DSecure": "boolean",
    "acceptedNetworks": ["visa", "mastercard", "mada"],
    "currency": "SAR"
  }
}
```

**Response:** `201 Created`
```json
{
  "id": "ch_xxx",
  "type": "card",
  "status": "active",
  "createdAt": "2026-02-08T15:00:00Z"
}
```

#### GET /channels

List all payment channels.

**Query Parameters:**
- `type`: Filter by channel type
- `status`: Filter by status (active, inactive)
- `limit`: Number of results (default: 20, max: 100)
- `offset`: Pagination offset

**Response:** `200 OK`
```json
{
  "data": [...],
  "hasMore": false,
  "total": 10
}
```

#### GET /channels/:id

Retrieve a specific channel.

**Response:** `200 OK`

#### PUT /channels/:id

Update channel configuration.

**Response:** `200 OK`

#### DELETE /channels/:id

Delete a payment channel.

**Response:** `200 OK`

### Payments API

#### POST /payments

Create a new payment.

**Request Body:**
```json
{
  "channelId": "ch_xxx",
  "amount": 100.00,
  "currency": "SAR",
  "description": "string",
  "customer": {
    "email": "string",
    "phone": "string",
    "name": "string"
  },
  "returnUrl": "string",
  "metadata": {}
}
```

**Response:** `201 Created`

#### GET /payments/:id

Retrieve payment details.

**Response:** `200 OK`

#### POST /payments/:id/refund

Refund a payment.

**Request Body:**
```json
{
  "amount": 50.00,
  "reason": "string"
}
```

**Response:** `200 OK`

#### POST /payments/:id/capture

Capture an authorized payment.

**Response:** `200 OK`

#### POST /payments/:id/cancel

Cancel a pending payment.

**Response:** `200 OK`

## Webhooks

### Configuration

Configure webhook endpoints in the Neoleap dashboard.

### Event Types

- `payment.created`
- `payment.succeeded`
- `payment.failed`
- `payment.refunded`
- `payment.disputed`
- `channel.created`
- `channel.updated`
- `channel.deleted`

### Payload Structure

```json
{
  "id": "evt_xxx",
  "type": "payment.succeeded",
  "createdAt": "2026-02-08T15:00:00Z",
  "data": {
    "id": "pay_xxx",
    "amount": 100.00,
    "status": "succeeded"
  }
}
```

## Rate Limits

- **Standard**: 100 requests per minute
- **Burst**: 1000 requests per hour
- **Payment Creation**: 10 per second

Rate limit headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1612800000
```

## Error Handling

### Error Response Format

```json
{
  "error": {
    "code": "invalid_request",
    "message": "Description of error",
    "param": "field_name",
    "type": "validation_error"
  }
}
```

### HTTP Status Codes

- `200` - Success
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `429` - Too Many Requests
- `500` - Internal Server Error

### Common Error Codes

- `authentication_failed` - Invalid credentials
- `invalid_request` - Missing or invalid parameters
- `resource_not_found` - Resource doesn't exist
- `rate_limit_exceeded` - Too many requests
- `insufficient_funds` - Payment declined
- `card_declined` - Card issuer declined

## SDKs

Official SDKs available for:
- Node.js
- Python
- PHP
- Ruby
- Java
- .NET

## Support

For API support, contact: api-support@neoleap.com
