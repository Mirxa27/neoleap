# Neoleap Integration Guide

## Overview

This guide walks you through integrating Neoleap payment channels into your application.

## Prerequisites

Before you begin, ensure you have:

1. **Neoleap Merchant Account**
   - Sign up at https://merchant.neoleap.com
   - Complete KYC verification
   - Obtain your API credentials

2. **Development Environment**
   - Server with HTTPS support
   - Supported programming language runtime
   - Webhook endpoint capability

3. **Technical Requirements**
   - SSL certificate for production
   - Ability to handle HTTP callbacks
   - JSON processing capability

## Integration Steps

### Step 1: Account Setup

1. **Create Merchant Account**
   ```
   Visit: https://merchant.neoleap.com/signup
   Complete: Business information and verification
   ```

2. **Get API Credentials**
   ```
   Navigate to: Dashboard > Settings > API Keys
   Generate: API Key and API Secret
   Note: Save these securely - they won't be shown again
   ```

3. **Configure Webhooks**
   ```
   Navigate to: Dashboard > Settings > Webhooks
   Add endpoint: https://yourdomain.com/webhooks/neoleap
   Select events: payment.succeeded, payment.failed
   ```

### Step 2: Install SDK

Choose your preferred programming language:

#### Node.js / JavaScript

```bash
npm install @neoleap/payment-sdk
```

#### Python

```bash
pip install neoleap-payment
```

#### PHP

```bash
composer require neoleap/payment-sdk
```

#### Ruby

```bash
gem install neoleap
```

### Step 3: Initialize SDK

#### Node.js Example

```javascript
const Neoleap = require('@neoleap/payment-sdk');

const client = new Neoleap({
  apiKey: process.env.NEOLEAP_API_KEY,
  apiSecret: process.env.NEOLEAP_API_SECRET,
  environment: 'sandbox' // Use 'production' when ready
});
```

#### Python Example

```python
import os
from neoleap import PaymentClient

client = PaymentClient(
    api_key=os.environ['NEOLEAP_API_KEY'],
    api_secret=os.environ['NEOLEAP_API_SECRET'],
    environment='sandbox'
)
```

#### PHP Example

```php
<?php
require 'vendor/autoload.php';

use Neoleap\PaymentClient;

$client = new PaymentClient([
    'apiKey' => $_ENV['NEOLEAP_API_KEY'],
    'apiSecret' => $_ENV['NEOLEAP_API_SECRET'],
    'environment' => 'sandbox'
]);
```

### Step 4: Create Payment Channel

Set up your first payment channel:

```javascript
const channel = await client.channels.create({
  type: 'card',
  name: 'Main Payment Channel',
  config: {
    enabled3DSecure: true,
    acceptedNetworks: ['visa', 'mastercard', 'mada'],
    currency: 'SAR'
  }
});

console.log('Channel ID:', channel.id);
// Save channel.id for future payments
```

### Step 5: Create Payment Form

#### Frontend HTML

```html
<!DOCTYPE html>
<html>
<head>
  <title>Checkout</title>
  <style>
    .payment-form {
      max-width: 400px;
      margin: 50px auto;
      padding: 20px;
      border: 1px solid #ddd;
      border-radius: 8px;
    }
    .form-group {
      margin-bottom: 15px;
    }
    label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    input {
      width: 100%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    button {
      width: 100%;
      padding: 12px;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    button:hover {
      background: #0056b3;
    }
  </style>
</head>
<body>
  <div class="payment-form">
    <h2>Complete Your Payment</h2>
    <form id="payment-form">
      <div class="form-group">
        <label>Amount (SAR)</label>
        <input type="number" id="amount" value="100.00" readonly>
      </div>
      <div class="form-group">
        <label>Email</label>
        <input type="email" id="email" required>
      </div>
      <div class="form-group">
        <label>Phone</label>
        <input type="tel" id="phone" required>
      </div>
      <button type="submit">Pay Now</button>
    </form>
  </div>

  <script>
    document.getElementById('payment-form').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const amount = document.getElementById('amount').value;
      const email = document.getElementById('email').value;
      const phone = document.getElementById('phone').value;
      
      try {
        const response = await fetch('/api/create-payment', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({ amount, email, phone })
        });
        
        const data = await response.json();
        
        if (data.paymentUrl) {
          // Redirect to payment page
          window.location.href = data.paymentUrl;
        }
      } catch (error) {
        alert('Payment failed: ' + error.message);
      }
    });
  </script>
</body>
</html>
```

#### Backend Endpoint

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.post('/api/create-payment', async (req, res) => {
  try {
    const { amount, email, phone } = req.body;
    
    // Validate input
    if (!amount || !email || !phone) {
      return res.status(400).json({ error: 'Missing required fields' });
    }
    
    // Create payment
    const payment = await client.payments.create({
      channelId: 'ch_your_channel_id', // Use your channel ID
      amount: parseFloat(amount),
      currency: 'SAR',
      description: 'Order payment',
      customer: {
        email: email,
        phone: phone
      },
      returnUrl: 'https://yourdomain.com/payment/complete',
      metadata: {
        source: 'web'
      }
    });
    
    res.json({
      paymentUrl: payment.paymentUrl,
      paymentId: payment.id
    });
  } catch (error) {
    console.error('Payment creation failed:', error);
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Step 6: Handle Payment Return

Create a page to handle customers returning from payment:

```javascript
app.get('/payment/complete', async (req, res) => {
  const paymentId = req.query.payment_id;
  
  try {
    // Retrieve payment status
    const payment = await client.payments.retrieve(paymentId);
    
    if (payment.status === 'succeeded') {
      res.send(`
        <h1>Payment Successful!</h1>
        <p>Thank you for your payment.</p>
        <p>Transaction ID: ${payment.id}</p>
      `);
    } else {
      res.send(`
        <h1>Payment Failed</h1>
        <p>Status: ${payment.status}</p>
      `);
    }
  } catch (error) {
    res.status(500).send('Error retrieving payment status');
  }
});
```

### Step 7: Implement Webhooks

Handle webhook notifications for real-time updates:

```javascript
app.post('/webhooks/neoleap', express.json(), async (req, res) => {
  const signature = req.headers['x-neoleap-signature'];
  const payload = req.body;
  
  // Verify webhook signature
  if (!client.webhooks.verify(payload, signature)) {
    return res.status(401).send('Invalid signature');
  }
  
  const event = payload;
  
  try {
    switch (event.type) {
      case 'payment.succeeded':
        // Update order status
        await updateOrderStatus(event.data.id, 'paid');
        // Send confirmation email
        await sendConfirmationEmail(event.data.customer.email);
        break;
        
      case 'payment.failed':
        // Handle failed payment
        await updateOrderStatus(event.data.id, 'failed');
        // Notify customer
        await notifyPaymentFailed(event.data.customer.email);
        break;
        
      case 'payment.refunded':
        // Process refund
        await processRefund(event.data.id);
        break;
    }
    
    res.status(200).send('OK');
  } catch (error) {
    console.error('Webhook processing error:', error);
    res.status(500).send('Error processing webhook');
  }
});
```

### Step 8: Testing

#### Use Sandbox Environment

1. **Test Credentials**
   - Use sandbox API keys for testing
   - No real money is processed

2. **Test Card Numbers**
   ```
   Success: 4111111111111111
   Decline: 4000000000000002
   3D Secure: 4000000000000101
   ```

3. **Test Scenarios**
   ```javascript
   // Successful payment
   const successPayment = await client.payments.create({
     channelId: channelId,
     amount: 100.00,
     currency: 'SAR',
     customer: { email: 'test@example.com' }
   });
   
   // Test refund
   await client.payments.refund(successPayment.id, {
     amount: 50.00
   });
   ```

### Step 9: Go Live

#### Pre-launch Checklist

- [ ] Complete merchant verification
- [ ] Switch to production API keys
- [ ] Update webhook URLs to production
- [ ] Configure SSL certificates
- [ ] Set up monitoring and logging
- [ ] Test production payment flow
- [ ] Enable fraud detection rules
- [ ] Set transaction limits

#### Switch to Production

```javascript
const productionClient = new Neoleap({
  apiKey: process.env.NEOLEAP_PRODUCTION_API_KEY,
  apiSecret: process.env.NEOLEAP_PRODUCTION_API_SECRET,
  environment: 'production'
});
```

### Step 10: Monitor and Maintain

1. **Monitor Transactions**
   - Dashboard: https://merchant.neoleap.com
   - View real-time transaction data
   - Track success rates
   - Monitor for anomalies

2. **Review Logs**
   ```javascript
   // Implement logging
   client.on('request', (req) => {
     logger.info('API Request:', req);
   });
   
   client.on('response', (res) => {
     logger.info('API Response:', res);
   });
   ```

3. **Handle Disputes**
   - Monitor dispute notifications
   - Respond with evidence
   - Update business processes

## Common Integration Patterns

### E-commerce Platform

```javascript
// Checkout flow
async function processCheckout(cart, customer) {
  const total = calculateTotal(cart.items);
  
  const payment = await client.payments.create({
    channelId: getChannelId(),
    amount: total,
    currency: 'SAR',
    description: `Order #${cart.id}`,
    customer: customer,
    metadata: {
      cartId: cart.id,
      items: cart.items.map(i => i.id)
    }
  });
  
  return payment.paymentUrl;
}
```

### Subscription Service

```javascript
// Recurring billing
async function processSubscription(customerId, planId) {
  const payment = await client.payments.create({
    channelId: getChannelId(),
    amount: getSubscriptionAmount(planId),
    currency: 'SAR',
    description: 'Monthly subscription',
    customer: { id: customerId },
    savePaymentMethod: true
  });
  
  // Store payment method for future charges
  await savePaymentMethod(customerId, payment.paymentMethod.id);
}
```

### Mobile App Integration

```javascript
// Mobile payment
async function createMobilePayment(orderId, amount, userId) {
  const payment = await client.payments.create({
    channelId: getChannelId(),
    amount: amount,
    currency: 'SAR',
    description: `Mobile order ${orderId}`,
    customer: {
      id: userId
    },
    returnUrl: `myapp://payment/complete?order=${orderId}`
  });
  
  return {
    paymentUrl: payment.paymentUrl,
    paymentId: payment.id
  };
}
```

## Security Best Practices

1. **API Key Management**
   - Store in environment variables
   - Never commit to version control
   - Rotate keys regularly
   - Use different keys per environment

2. **Webhook Security**
   - Always verify signatures
   - Use HTTPS endpoints
   - Implement rate limiting
   - Log all events

3. **PCI Compliance**
   - Never store card details
   - Use tokenization
   - Implement 3D Secure
   - Follow PCI DSS guidelines

## Support and Resources

- **Documentation**: https://docs.neoleap.com
- **Support Email**: support@neoleap.com
- **Developer Forum**: https://community.neoleap.com
- **API Status**: https://status.neoleap.com

## Next Steps

1. Explore advanced features (recurring payments, multi-currency)
2. Implement additional payment channels (wallets, bank transfers)
3. Set up fraud detection rules
4. Optimize payment flow for conversions
5. Review analytics and insights

---

**Need Help?** Contact our integration support team at integration@neoleap.com
