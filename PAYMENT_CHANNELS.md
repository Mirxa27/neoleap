# Neoleap Payment Channels Documentation

## Table of Contents
1. [Overview](#overview)
2. [Payment Channel Types](#payment-channel-types)
3. [Architecture](#architecture)
4. [Implementation](#implementation)
5. [API Reference](#api-reference)
6. [Security](#security)
7. [Best Practices](#best-practices)
8. [Examples](#examples)
9. [Troubleshooting](#troubleshooting)

## Overview

Payment channels in Neoleap provide a secure, efficient, and scalable method for processing digital payments. A payment channel is a communication pathway between payment parties that enables the transfer of funds while maintaining security, compliance, and reliability.

### Key Features

- **Real-time Processing**: Instant payment confirmations and settlements
- **Multi-channel Support**: Cards, digital wallets, bank transfers, and more
- **Secure Transactions**: End-to-end encryption and PCI DSS compliance
- **Scalable Infrastructure**: Handle high transaction volumes
- **Comprehensive Reporting**: Detailed transaction logs and analytics

### Benefits

- Reduced transaction costs
- Faster settlement times
- Enhanced security measures
- Seamless integration capabilities
- Support for multiple payment methods

## Payment Channel Types

Neoleap supports multiple payment channel types to accommodate various business needs:

### 1. Card Payment Channels

Process credit and debit card transactions through various networks.

**Supported Networks:**
- Visa
- Mastercard
- Mada (Saudi Arabia domestic)
- American Express

**Features:**
- 3D Secure authentication
- Tokenization for recurring payments
- Support for contactless payments
- EMV chip card processing

### 2. Digital Wallet Channels

Integration with popular digital wallet providers.

**Supported Wallets:**
- Apple Pay
- Google Pay
- Samsung Pay
- STC Pay
- Neoleap Wallet

**Features:**
- One-click checkout
- Biometric authentication
- QR code payments
- P2P transfers

### 3. Bank Transfer Channels

Direct bank-to-bank payment processing.

**Types:**
- SADAD (Saudi Arabia)
- SWIFT transfers
- Local ACH transfers
- Real-time bank transfers (SARIE)

**Features:**
- Batch processing support
- Scheduled payments
- Automated reconciliation
- Multi-currency support

### 4. Point of Sale (POS) Channels

In-person payment processing for retail environments.

**Capabilities:**
- Integrated POS terminals
- Mobile POS solutions
- Tap-to-pay functionality
- Receipt management

## Architecture

### Channel Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                  Client Application                  │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│              Neoleap Payment Gateway                 │
│  ┌──────────────────────────────────────────────┐  │
│  │         Channel Selection Engine              │  │
│  └──────────────────┬───────────────────────────┘  │
│                     │                                │
│     ┌───────────────┼───────────────┐               │
│     ▼               ▼               ▼               │
│  ┌─────┐      ┌─────────┐     ┌────────┐          │
│  │Card │      │ Wallet  │     │  Bank  │          │
│  │Chan.│      │ Channel │     │Channel │          │
│  └──┬──┘      └────┬────┘     └───┬────┘          │
└─────┼──────────────┼──────────────┼───────────────┘
      │              │              │
      ▼              ▼              ▼
┌─────────┐   ┌──────────┐   ┌──────────┐
│Payment  │   │ Wallet   │   │  Bank    │
│Networks │   │Providers │   │ Systems  │
└─────────┘   └──────────┘   └──────────┘
```

### Key Components

1. **Payment Gateway**: Central hub for routing transactions
2. **Channel Selector**: Determines optimal payment channel based on transaction type
3. **Authentication Service**: Handles security and authorization
4. **Settlement Engine**: Manages fund transfers and reconciliation
5. **Reporting Service**: Generates transaction reports and analytics

## Implementation

### Getting Started

#### Prerequisites

- Neoleap merchant account
- API credentials (API Key and Secret)
- SSL certificate for secure communication
- Webhook endpoint for payment notifications

#### Installation

For Node.js:
```bash
npm install @neoleap/payment-sdk
```

For Python:
```bash
pip install neoleap-payment
```

For PHP:
```bash
composer require neoleap/payment-sdk
```

### Basic Integration

#### 1. Initialize the SDK

**Node.js:**
```javascript
const Neoleap = require('@neoleap/payment-sdk');

const client = new Neoleap({
  apiKey: 'your_api_key',
  apiSecret: 'your_api_secret',
  environment: 'production' // or 'sandbox'
});
```

**Python:**
```python
from neoleap import PaymentClient

client = PaymentClient(
    api_key='your_api_key',
    api_secret='your_api_secret',
    environment='production'  # or 'sandbox'
)
```

#### 2. Create a Payment Channel

```javascript
// Card payment channel
const cardChannel = await client.channels.create({
  type: 'card',
  config: {
    enabled3DSecure: true,
    acceptedNetworks: ['visa', 'mastercard', 'mada'],
    currency: 'SAR'
  }
});

// Wallet payment channel
const walletChannel = await client.channels.create({
  type: 'wallet',
  config: {
    providers: ['applepay', 'googlepay', 'stcpay'],
    currency: 'SAR'
  }
});
```

#### 3. Process a Payment

```javascript
const payment = await client.payments.create({
  channelId: cardChannel.id,
  amount: 100.00,
  currency: 'SAR',
  description: 'Order #12345',
  customer: {
    email: 'customer@example.com',
    phone: '+966501234567'
  },
  metadata: {
    orderId: '12345',
    source: 'web'
  }
});

// Redirect customer to payment page
console.log('Payment URL:', payment.paymentUrl);
```

#### 4. Handle Webhooks

```javascript
const express = require('express');
const app = express();

app.post('/webhooks/neoleap', express.json(), (req, res) => {
  const signature = req.headers['x-neoleap-signature'];
  
  // Verify webhook signature
  if (!client.webhooks.verify(req.body, signature)) {
    return res.status(401).send('Invalid signature');
  }
  
  const event = req.body;
  
  switch (event.type) {
    case 'payment.succeeded':
      console.log('Payment succeeded:', event.data);
      // Update order status
      break;
    case 'payment.failed':
      console.log('Payment failed:', event.data);
      // Handle failure
      break;
    case 'payment.refunded':
      console.log('Payment refunded:', event.data);
      // Process refund
      break;
  }
  
  res.status(200).send('OK');
});
```

## API Reference

### Channels

#### Create Channel

**Endpoint:** `POST /api/v1/channels`

**Request:**
```json
{
  "type": "card",
  "name": "Main Card Channel",
  "config": {
    "enabled3DSecure": true,
    "acceptedNetworks": ["visa", "mastercard"],
    "currency": "SAR"
  }
}
```

**Response:**
```json
{
  "id": "ch_1234567890",
  "type": "card",
  "name": "Main Card Channel",
  "status": "active",
  "config": {
    "enabled3DSecure": true,
    "acceptedNetworks": ["visa", "mastercard"],
    "currency": "SAR"
  },
  "createdAt": "2026-02-08T15:00:00Z"
}
```

#### List Channels

**Endpoint:** `GET /api/v1/channels`

**Query Parameters:**
- `type` (optional): Filter by channel type
- `status` (optional): Filter by status (active, inactive)
- `limit` (optional): Number of results (default: 20)
- `offset` (optional): Pagination offset

**Response:**
```json
{
  "data": [
    {
      "id": "ch_1234567890",
      "type": "card",
      "name": "Main Card Channel",
      "status": "active"
    }
  ],
  "hasMore": false,
  "total": 1
}
```

#### Update Channel

**Endpoint:** `PUT /api/v1/channels/:id`

**Request:**
```json
{
  "name": "Updated Channel Name",
  "config": {
    "enabled3DSecure": false
  }
}
```

#### Delete Channel

**Endpoint:** `DELETE /api/v1/channels/:id`

**Response:**
```json
{
  "id": "ch_1234567890",
  "deleted": true
}
```

### Payments

#### Create Payment

**Endpoint:** `POST /api/v1/payments`

**Request:**
```json
{
  "channelId": "ch_1234567890",
  "amount": 100.00,
  "currency": "SAR",
  "description": "Product purchase",
  "customer": {
    "email": "customer@example.com",
    "phone": "+966501234567",
    "name": "Ahmad Ali"
  },
  "returnUrl": "https://yoursite.com/payment/return",
  "metadata": {
    "orderId": "12345"
  }
}
```

**Response:**
```json
{
  "id": "pay_9876543210",
  "channelId": "ch_1234567890",
  "amount": 100.00,
  "currency": "SAR",
  "status": "pending",
  "paymentUrl": "https://payment.neoleap.com/pay/9876543210",
  "createdAt": "2026-02-08T15:30:00Z"
}
```

#### Retrieve Payment

**Endpoint:** `GET /api/v1/payments/:id`

**Response:**
```json
{
  "id": "pay_9876543210",
  "channelId": "ch_1234567890",
  "amount": 100.00,
  "currency": "SAR",
  "status": "succeeded",
  "customer": {
    "email": "customer@example.com"
  },
  "paymentMethod": {
    "type": "card",
    "last4": "4242",
    "brand": "visa"
  },
  "createdAt": "2026-02-08T15:30:00Z",
  "completedAt": "2026-02-08T15:31:00Z"
}
```

#### Refund Payment

**Endpoint:** `POST /api/v1/payments/:id/refund`

**Request:**
```json
{
  "amount": 50.00,
  "reason": "Customer request"
}
```

**Response:**
```json
{
  "id": "ref_1122334455",
  "paymentId": "pay_9876543210",
  "amount": 50.00,
  "status": "succeeded",
  "createdAt": "2026-02-08T16:00:00Z"
}
```

### Webhook Events

Neoleap sends webhook events for important payment activities.

**Event Types:**
- `channel.created` - New channel created
- `channel.updated` - Channel configuration updated
- `channel.deleted` - Channel deleted
- `payment.created` - New payment initiated
- `payment.succeeded` - Payment completed successfully
- `payment.failed` - Payment failed
- `payment.refunded` - Payment refunded
- `payment.disputed` - Payment disputed by customer

**Webhook Payload:**
```json
{
  "id": "evt_1234567890",
  "type": "payment.succeeded",
  "createdAt": "2026-02-08T15:31:00Z",
  "data": {
    "id": "pay_9876543210",
    "amount": 100.00,
    "currency": "SAR",
    "status": "succeeded"
  }
}
```

## Security

### Authentication

All API requests must include authentication headers:

```
Authorization: Bearer {API_KEY}
X-Neoleap-Signature: {HMAC_SIGNATURE}
```

### HMAC Signature Generation

```javascript
const crypto = require('crypto');

function generateSignature(payload, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(JSON.stringify(payload));
  return hmac.digest('hex');
}
```

### PCI DSS Compliance

Neoleap is PCI DSS Level 1 compliant. When handling card data:

1. **Never store card numbers** - Use tokenization
2. **Use HTTPS** - All communications must be encrypted
3. **Implement 3D Secure** - For additional cardholder authentication
4. **Validate input** - Sanitize all user inputs
5. **Monitor transactions** - Set up fraud detection alerts

### Data Encryption

- **In Transit**: TLS 1.2+ encryption for all API communications
- **At Rest**: AES-256 encryption for stored payment data
- **Tokenization**: Card details replaced with secure tokens

### Best Security Practices

1. **API Key Management**
   - Store keys in environment variables
   - Never commit keys to version control
   - Rotate keys periodically
   - Use different keys for sandbox and production

2. **Webhook Security**
   - Always verify webhook signatures
   - Use HTTPS endpoints only
   - Implement idempotency checks
   - Log all webhook events

3. **Transaction Monitoring**
   - Set velocity limits
   - Monitor for unusual patterns
   - Implement fraud detection rules
   - Enable real-time alerts

## Best Practices

### 1. Channel Selection

Choose the appropriate payment channel based on:
- **Geography**: Use Mada for Saudi domestic cards
- **User preference**: Offer popular local wallets
- **Transaction amount**: Consider fees for different channels
- **Settlement speed**: Bank transfers vs instant payments

### 2. Error Handling

Implement robust error handling:

```javascript
try {
  const payment = await client.payments.create(paymentData);
  // Handle success
} catch (error) {
  if (error.code === 'insufficient_funds') {
    // Show appropriate message
  } else if (error.code === 'card_declined') {
    // Suggest alternative payment method
  } else if (error.code === 'authentication_failed') {
    // Request 3D Secure authentication
  } else {
    // Generic error handling
    console.error('Payment error:', error.message);
  }
}
```

### 3. User Experience

- **Payment Method Display**: Show logos of supported payment methods
- **Loading States**: Indicate when processing payments
- **Clear Communication**: Provide status updates
- **Error Messages**: Use user-friendly language
- **Mobile Optimization**: Ensure responsive design

### 4. Testing

Always test in sandbox environment:

```javascript
const sandboxClient = new Neoleap({
  apiKey: 'sandbox_api_key',
  apiSecret: 'sandbox_api_secret',
  environment: 'sandbox'
});

// Use test card numbers
const testPayment = {
  channelId: 'ch_test',
  amount: 100.00,
  cardNumber: '4111111111111111', // Test Visa card
  cvv: '123',
  expiryMonth: '12',
  expiryYear: '2027'
};
```

**Test Card Numbers:**
- Visa: `4111111111111111`
- Mastercard: `5500000000000004`
- Mada: `5297410000000000`

### 5. Performance Optimization

- **Caching**: Cache channel configurations
- **Async Processing**: Use webhooks for status updates
- **Retry Logic**: Implement exponential backoff
- **Connection Pooling**: Reuse HTTP connections
- **Batch Operations**: Group multiple operations when possible

## Examples

### Example 1: E-commerce Checkout

```javascript
const express = require('express');
const Neoleap = require('@neoleap/payment-sdk');

const app = express();
const client = new Neoleap({
  apiKey: process.env.NEOLEAP_API_KEY,
  apiSecret: process.env.NEOLEAP_API_SECRET
});

app.post('/checkout', async (req, res) => {
  try {
    const { items, customer } = req.body;
    
    // Calculate total
    const total = items.reduce((sum, item) => sum + item.price, 0);
    
    // Create payment
    const payment = await client.payments.create({
      channelId: 'ch_main_channel',
      amount: total,
      currency: 'SAR',
      description: `Order for ${customer.email}`,
      customer: customer,
      returnUrl: 'https://yourstore.com/payment/complete',
      metadata: {
        items: items.map(i => i.id),
        cartId: req.session.cartId
      }
    });
    
    res.json({ paymentUrl: payment.paymentUrl });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Example 2: Subscription Payment

```javascript
async function createSubscription(customerId, planId) {
  // Create channel for recurring payments
  const channel = await client.channels.create({
    type: 'card',
    config: {
      recurringEnabled: true,
      tokenizationEnabled: true
    }
  });
  
  // Create initial payment and save token
  const payment = await client.payments.create({
    channelId: channel.id,
    amount: 99.00,
    currency: 'SAR',
    description: 'Subscription - Monthly Plan',
    customer: { id: customerId },
    savePaymentMethod: true
  });
  
  // Store payment method token for future charges
  const paymentMethod = payment.paymentMethod.id;
  
  return {
    subscriptionId: generateSubscriptionId(),
    paymentMethodToken: paymentMethod,
    nextBillingDate: getNextBillingDate()
  };
}
```

### Example 3: Mobile Wallet Payment

```javascript
async function processWalletPayment(walletType, phoneNumber, amount) {
  // Create wallet channel
  const walletChannel = await client.channels.create({
    type: 'wallet',
    config: {
      providers: [walletType] // 'applepay', 'googlepay', 'stcpay'
    }
  });
  
  // Create payment
  const payment = await client.payments.create({
    channelId: walletChannel.id,
    amount: amount,
    currency: 'SAR',
    customer: {
      phone: phoneNumber
    },
    paymentMethod: {
      type: 'wallet',
      wallet: walletType
    }
  });
  
  return payment;
}
```

### Example 4: Refund Processing

```javascript
async function processRefund(paymentId, refundAmount, reason) {
  try {
    // Get original payment
    const payment = await client.payments.retrieve(paymentId);
    
    if (payment.status !== 'succeeded') {
      throw new Error('Cannot refund non-successful payment');
    }
    
    // Process refund
    const refund = await client.payments.refund(paymentId, {
      amount: refundAmount || payment.amount, // Full or partial refund
      reason: reason,
      metadata: {
        processedBy: 'admin',
        timestamp: new Date().toISOString()
      }
    });
    
    // Notify customer
    await notifyCustomer(payment.customer.email, {
      type: 'refund',
      amount: refund.amount,
      paymentId: paymentId
    });
    
    return refund;
  } catch (error) {
    console.error('Refund failed:', error);
    throw error;
  }
}
```

## Troubleshooting

### Common Issues

#### 1. Authentication Errors

**Problem**: `401 Unauthorized` response

**Solutions:**
- Verify API key and secret are correct
- Check if using correct environment (sandbox vs production)
- Ensure API key is active and not expired
- Verify signature generation algorithm

#### 2. Payment Declined

**Problem**: Card payment declined

**Solutions:**
- Check card details are correct
- Verify sufficient funds
- Ensure 3D Secure is completed
- Check if card is supported in region
- Review fraud detection rules

#### 3. Webhook Not Received

**Problem**: Webhook events not arriving

**Solutions:**
- Verify webhook URL is accessible
- Check SSL certificate is valid
- Ensure endpoint returns 200 status
- Review firewall settings
- Check webhook configuration in dashboard

#### 4. Channel Creation Failed

**Problem**: Cannot create payment channel

**Solutions:**
- Verify merchant account is activated
- Check if channel type is supported
- Ensure configuration is valid
- Review account limits and restrictions

#### 5. Slow Transaction Processing

**Problem**: Payments taking too long

**Solutions:**
- Check network connectivity
- Verify API endpoint region
- Review transaction volume limits
- Implement connection pooling
- Use async processing where possible

### Error Codes

| Code | Description | Resolution |
|------|-------------|------------|
| `authentication_failed` | Invalid API credentials | Check API key and secret |
| `insufficient_funds` | Not enough balance | Request customer to use another card |
| `card_declined` | Card issuer declined | Suggest alternative payment method |
| `expired_card` | Card has expired | Update card details |
| `invalid_cvv` | CVV verification failed | Re-enter CVV code |
| `channel_not_found` | Channel ID invalid | Verify channel exists |
| `amount_too_small` | Below minimum amount | Increase transaction amount |
| `amount_too_large` | Exceeds maximum amount | Split into multiple transactions |
| `unsupported_currency` | Currency not supported | Use supported currency |
| `rate_limit_exceeded` | Too many requests | Implement rate limiting |

### Debug Mode

Enable debug logging:

```javascript
const client = new Neoleap({
  apiKey: 'your_api_key',
  apiSecret: 'your_api_secret',
  debug: true // Enable debug mode
});

// View request/response logs
client.on('request', (req) => {
  console.log('Request:', req);
});

client.on('response', (res) => {
  console.log('Response:', res);
});
```

### Support Resources

- **Documentation**: https://docs.neoleap.com
- **API Status**: https://status.neoleap.com
- **Support Email**: support@neoleap.com
- **Developer Forum**: https://community.neoleap.com

## Conclusion

Neoleap payment channels provide a comprehensive, secure, and scalable solution for digital payments. This documentation covers the essential aspects of implementing and managing payment channels. For additional support or advanced use cases, please contact the Neoleap technical team.

---

**Document Version**: 1.0  
**Last Updated**: February 8, 2026  
**Maintained by**: Neoleap Technical Team
