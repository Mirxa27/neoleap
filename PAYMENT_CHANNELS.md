# Al Rajhi Bank (ARB) Payment Gateway Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Integration Types](#integration-types)
4. [Transaction Flows](#transaction-flows)
5. [API Specifications](#api-specifications)
6. [Security & Encryption](#security--encryption)
7. [Payment Methods](#payment-methods)
8. [Testing](#testing)
9. [Troubleshooting](#troubleshooting)

## Introduction

### About ARB Payment Gateway

Al Rajhi Bank Payment Gateway is a secure, PCI-DSS compliant payment processing solution that enables merchants to accept online payments from customers. The gateway supports multiple payment methods including credit/debit cards (Visa, Mastercard, MADA), digital wallets (Apple Pay), URPay, SADAD, and installment plans.

The ARB Payment Gateway provides merchants with a comprehensive payment solution that includes:

- **Secure Payment Processing**: PCI-DSS Level 1 certified infrastructure
- **Multiple Integration Options**: Bank-hosted and merchant-hosted solutions
- **3D Secure Authentication**: Enhanced security for card transactions
- **Real-time Transaction Processing**: Instant payment authorization and settlement
- **Comprehensive Reporting**: Detailed transaction reports and reconciliation tools
- **Multi-currency Support**: Process payments in multiple currencies
- **Fraud Prevention**: Advanced fraud detection and prevention mechanisms

### PCI-DSS Compliance

Al Rajhi Bank Payment Gateway is PCI-DSS Level 1 compliant, ensuring the highest level of security for payment card data. The gateway follows strict security standards including:

- Secure network infrastructure with firewall protection
- Encryption of cardholder data during transmission and storage
- Regular security testing and monitoring
- Access control measures and activity logging
- Secure application development practices

Merchants integrating with ARB Payment Gateway benefit from this compliance without needing to maintain their own PCI-DSS certification for basic integrations.

### Target Audience

This documentation is intended for:

- **Developers**: Technical teams implementing payment gateway integration
- **System Architects**: Designing payment processing systems
- **Business Analysts**: Understanding payment flows and capabilities
- **QA Engineers**: Testing payment implementations
- **Support Teams**: Troubleshooting payment issues

## Getting Started

### Prerequisites

Before integrating with ARB Payment Gateway, you need:

1. **Merchant Account**: Active merchant account with Al Rajhi Bank
2. **Tranportal Credentials**:
   - Tranportal ID (Merchant ID)
   - Tranportal Password
   - Resource Key (for encryption)
3. **Technical Requirements**:
   - SSL certificate for production environment
   - Server with HTTPS support
   - Ability to handle redirect/callback URLs
4. **Development Environment**: Access to sandbox environment for testing

### Obtaining Credentials

To obtain your Tranportal credentials:

1. Contact Al Rajhi Bank merchant services
2. Complete merchant onboarding process
3. Receive Tranportal ID and Password
4. Obtain Resource Key for encryption
5. Get sandbox credentials for testing

### Endpoints

#### Sandbox Environment
```
Payment URL: https://test.arb.gateway.mastercard.com/checkout/version/XX/checkout.js
Payment API: https://test.arb.gateway.mastercard.com/api/rest/version/XX/merchant/{merchantId}
Tranportal: https://sandbox.tranportal.alrajhibank.com.sa
```

#### Production Environment
```
Payment URL: https://arb.gateway.mastercard.com/checkout/version/XX/checkout.js
Payment API: https://arb.gateway.mastercard.com/api/rest/version/XX/merchant/{merchantId}
Tranportal: https://tranportal.alrajhibank.com.sa
```

### Environment Setup

**1. Register Your Domain**
- Whitelist your domain/IP in Tranportal
- Configure return URLs
- Set webhook endpoints

**2. Configure Security Settings**
- Upload SSL certificate
- Set password policies
- Enable 3D Secure
- Configure fraud rules

**3. Test Connectivity**
```bash
# Test API endpoint connectivity
curl -X GET "https://test.arb.gateway.mastercard.com/api/rest/version/XX/merchant/{merchantId}/information" \
  -u "merchant.{merchantId}:password"
```

## Integration Types

ARB Payment Gateway offers multiple integration methods to suit different merchant requirements:

### 1. Bank Hosted Integration

The simplest integration where the payment page is hosted by Al Rajhi Bank.

#### Standard Payment Page

**Features:**
- Fully hosted payment page
- No PCI compliance required for merchant
- Customizable with merchant branding
- Automatic updates and security patches

**Use Cases:**
- Quick integration requirements
- Merchants without technical resources
- Minimal PCI compliance requirements

**Integration Steps:**
1. Generate payment token via API
2. Redirect customer to ARB payment page
3. Customer completes payment
4. ARB redirects back to merchant site
5. Merchant queries transaction status

#### Faster Checkout

**Features:**
- Streamlined payment flow
- Auto-filled customer information
- Reduced checkout steps
- Mobile-optimized interface

**Benefits:**
- Higher conversion rates
- Better user experience
- Reduced cart abandonment
- Quick payment processing

#### iFrame Integration

**Features:**
- Payment form embedded in merchant page
- Seamless user experience
- Merchant maintains page control
- PCI scope remains with ARB

**Implementation:**
```html
<iframe 
  id="arbPaymentFrame"
  src="https://arb.gateway.mastercard.com/checkout/version/XX/checkout.js"
  width="100%"
  height="500px"
  frameborder="0">
</iframe>
```

### 2. Merchant Hosted Integration

Advanced integration where merchant hosts the payment form with full control over UX.

#### 3D Secure (Recommended)

**Features:**
- Additional authentication layer
- Reduced fraud and chargebacks
- Liability shift to issuing bank
- Supports Visa Secure and Mastercard SecureCode

**Flow:**
1. Customer enters card details on merchant page
2. Merchant encrypts and sends to ARB
3. ARB initiates 3D Secure authentication
4. Customer completes authentication with issuer
5. Transaction processed after successful auth

**Implementation:**
```javascript
// Encrypt card data
const encryptedData = encryptCardData({
  cardNumber: '4111111111111111',
  expiryMonth: '12',
  expiryYear: '2025',
  cvv: '123'
}, resourceKey);

// Submit to ARB
const response = await processPayment({
  merchantId: 'YOUR_MERCHANT_ID',
  amount: '100.00',
  currency: 'SAR',
  encryptedCardData: encryptedData,
  secure3D: true
});
```

#### Non-3D Secure

**Features:**
- Direct payment processing
- Faster transaction flow
- No customer redirection
- Higher merchant liability

**Use Cases:**
- Low-risk transactions
- Trusted customer base
- Business-to-business payments
- Subscription renewals

**Risk Considerations:**
- Merchant bears chargeback liability
- Higher fraud risk
- May have lower authorization rates
- Recommended to implement additional fraud checks

### 3. Card-on-File (Tokenization)

**Features:**
- Store payment methods securely
- Process recurring payments
- One-click checkout
- PCI-compliant token storage

**Use Cases:**
- Subscription services
- Recurring billing
- Quick re-ordering
- Membership fees

**Token Generation:**
```javascript
// Initial transaction with tokenization
const tokenResponse = await createPayment({
  merchantId: 'YOUR_MERCHANT_ID',
  amount: '100.00',
  currency: 'SAR',
  cardData: encryptedCardData,
  storeCard: true,
  customerId: 'CUST_12345'
});

// Token stored: tokenResponse.token
```

**Subsequent Transactions:**
```javascript
// Use token for future payments
const payment = await processTokenPayment({
  merchantId: 'YOUR_MERCHANT_ID',
  amount: '100.00',
  currency: 'SAR',
  token: 'STORED_TOKEN',
  customerId: 'CUST_12345'
});
```

### Integration Comparison

| Feature | Bank Hosted | Merchant Hosted (3DS) | Merchant Hosted (Non-3DS) | Card-on-File |
|---------|-------------|------------------------|----------------------------|--------------|
| PCI Compliance | Minimal | Moderate | High | Minimal |
| Customization | Limited | Full | Full | Full |
| Fraud Protection | High | High | Moderate | High |
| Integration Effort | Low | Moderate | Moderate | High |
| User Experience | Good | Excellent | Excellent | Excellent |
| Chargeback Liability | Shared | Issuer | Merchant | Issuer* |

*When using 3DS for initial tokenization

## Transaction Flows

### Bank Hosted - Standard Flow

```
Customer → Merchant → ARB Gateway → Payment Page → Customer Auth → Processing → Result → Merchant
```

**Detailed Steps:**

1. **Initiate Payment**
   - Customer clicks "Pay" on merchant website
   - Merchant prepares transaction details

2. **Generate Payment Token**
   ```javascript
   POST /api/generateToken
   {
     "merchantId": "M123456",
     "amount": "100.00",
     "currency": "SAR",
     "orderId": "ORD-2024-001",
     "returnUrl": "https://merchant.com/return"
   }
   ```

3. **Redirect to Payment Page**
   - Merchant redirects customer to ARB payment page with token
   - URL: `https://arb.gateway.com/checkout?token=PAYMENT_TOKEN`

4. **Customer Enters Payment Details**
   - Customer enters card number, expiry, CVV
   - Selects payment method (Card/Apple Pay/etc.)

5. **3D Secure Authentication** (if applicable)
   - Customer redirected to issuing bank
   - Completes OTP or biometric authentication
   - Returns to ARB gateway

6. **Transaction Processing**
   - ARB gateway sends authorization request
   - Issuing bank approves/declines
   - Transaction recorded

7. **Return to Merchant**
   - Customer redirected to merchant returnUrl
   - Merchant queries transaction status via API

8. **Status Verification**
   ```javascript
   GET /api/transaction/{transactionId}
   Response: {
     "status": "SUCCESS",
     "transactionId": "TXN123456",
     "amount": "100.00",
     "authCode": "123456"
   }
   ```

### Merchant Hosted - 3D Secure Flow

```
Customer → Merchant Form → Encrypt Data → ARB API → 3DS Auth → Issuer → Complete → Merchant
```

**Detailed Steps:**

1. **Customer Enters Card Details**
   - On merchant's payment form
   - Card number, expiry, CVV, name

2. **Client-Side Validation**
   ```javascript
   // Validate card number
   function validateCard(cardNumber) {
     // Luhn algorithm check
     // Card type detection
     return isValid;
   }
   ```

3. **Encrypt Card Data**
   ```javascript
   const encryptedData = encryptAES({
     cardNumber: cardNumber,
     expiryMonth: expiryMonth,
     expiryYear: expiryYear,
     cvv: cvv
   }, resourceKey, iv);
   ```

4. **Submit to Merchant Server**
   - Encrypted data sent via HTTPS POST
   - Server-side validation

5. **Call ARB Payment API**
   ```javascript
   POST /api/rest/version/XX/merchant/{merchantId}/order/{orderId}/transaction/1
   Authorization: Basic {base64(merchant.merchantId:password)}
   {
     "apiOperation": "INITIATE_AUTHENTICATION",
     "authentication": {
       "channel": "PAYER_BROWSER",
       "purpose": "PAYMENT_TRANSACTION"
     },
     "order": {
       "amount": "100.00",
       "currency": "SAR"
     },
     "sourceOfFunds": {
       "type": "CARD",
       "provided": {
         "card": {
           "number": "{encrypted}",
           "securityCode": "{encrypted}",
           "expiry": {
             "month": "12",
             "year": "25"
           }
         }
       }
     }
   }
   ```

6. **3D Secure Redirect**
   - ARB returns authentication URL
   - Merchant redirects customer to issuer
   - Display in iframe or popup

7. **Customer Authentication**
   - Customer completes OTP/password/biometric
   - Issuer validates authentication

8. **Authentication Complete**
   - Customer redirected back to merchant
   - Authentication result received

9. **Authenticate Payment**
   ```javascript
   POST /api/rest/version/XX/merchant/{merchantId}/order/{orderId}/transaction/2
   {
     "apiOperation": "AUTHENTICATE_PAYER",
     "authentication": {
       "transactionId": "{3DS_TransactionId}"
     }
   }
   ```

10. **Process Payment**
    ```javascript
    POST /api/rest/version/XX/merchant/{merchantId}/order/{orderId}/transaction/3
    {
      "apiOperation": "PAY",
      "authentication": {
        "transactionId": "{3DS_TransactionId}"
      },
      "order": {
        "amount": "100.00",
        "currency": "SAR"
      }
    }
    ```

11. **Display Result**
    - Show success/failure to customer
    - Send confirmation email
    - Update order status

### Card-on-File Flow

**Initial Setup (Tokenization):**

```
Customer → Card Entry → 3DS Auth → Token Creation → Store Token → Confirmation
```

1. **First Payment with Tokenization**
   ```javascript
   POST /api/createToken
   {
     "merchantId": "M123456",
     "customerId": "CUST_001",
     "amount": "100.00",
     "currency": "SAR",
     "cardData": "{encrypted}",
     "storeCard": true,
     "3dsRequired": true
   }
   ```

2. **Complete 3DS Authentication**
   - First-time tokenization requires authentication
   - Customer completes issuer verification

3. **Token Stored**
   ```javascript
   Response: {
     "status": "SUCCESS",
     "transactionId": "TXN123456",
     "token": "TKN_ABC123XYZ",
     "cardBrand": "VISA",
     "cardLast4": "1111",
     "expiryMonth": "12",
     "expiryYear": "2025"
   }
   ```

**Subsequent Payments:**

```
Merchant System → Token Payment API → Processing → Result → Update System
```

1. **Process Token Payment**
   ```javascript
   POST /api/processTokenPayment
   {
     "merchantId": "M123456",
     "customerId": "CUST_001",
     "token": "TKN_ABC123XYZ",
     "amount": "150.00",
     "currency": "SAR",
     "orderId": "ORD-2024-002",
     "description": "Subscription renewal"
   }
   ```

2. **Authorization**
   - No customer interaction required
   - Automatic processing
   - Instant result

3. **Result Notification**
   ```javascript
   Response: {
     "status": "SUCCESS",
     "transactionId": "TXN123457",
     "authCode": "654321",
     "amount": "150.00"
   }
   ```

### Apple Pay Flow

```
Customer → Apple Pay Button → Wallet Auth → Payment Sheet → Token → ARB → Processing → Result
```

**Integration Steps:**

1. **Configure Apple Pay**
   - Register merchant identifier with Apple
   - Create payment processing certificate
   - Verify domain with Apple

2. **Display Apple Pay Button**
   ```javascript
   if (window.ApplePaySession && 
       ApplePaySession.canMakePayments()) {
     // Show Apple Pay button
     showApplePayButton();
   }
   ```

3. **Create Payment Request**
   ```javascript
   const request = {
     countryCode: 'SA',
     currencyCode: 'SAR',
     supportedNetworks: ['visa', 'masterCard', 'mada'],
     merchantCapabilities: ['supports3DS'],
     total: {
       label: 'Merchant Name',
       amount: '100.00'
     }
   };
   ```

4. **Start Apple Pay Session**
   ```javascript
   const session = new ApplePaySession(3, request);
   
   session.onvalidatemerchant = async (event) => {
     // Validate with merchant server
     const merchantSession = await validateMerchant(event.validationURL);
     session.completeMerchantValidation(merchantSession);
   };
   ```

5. **Process Payment**
   ```javascript
   session.onpaymentauthorized = async (event) => {
     const payment = event.payment;
     const result = await processApplePayment(payment.token);
     
     if (result.success) {
       session.completePayment(ApplePaySession.STATUS_SUCCESS);
     } else {
       session.completePayment(ApplePaySession.STATUS_FAILURE);
     }
   };
   ```

6. **Submit to ARB**
   ```javascript
   POST /api/applePayment
   {
     "merchantId": "M123456",
     "amount": "100.00",
     "currency": "SAR",
     "paymentData": {
       "version": "EC_v1",
       "data": "{encrypted_payment_data}",
       "signature": "{signature}",
       "header": {
         "publicKeyHash": "{hash}",
         "ephemeralPublicKey": "{key}",
         "transactionId": "{txn_id}"
       }
     }
   }
   ```

## API Specifications

### Authentication

All API requests require HTTP Basic Authentication:

```
Authorization: Basic {base64(merchant.{merchantId}:{password})}
```

**Example:**
```bash
# MerchantId: M123456
# Password: myPassword123
# Base64 encode: merchant.M123456:myPassword123

Authorization: Basic bWVyY2hhbnQuTTEyMzQ1NjpteVBhc3N3b3JkMTIz
```

### Payment Token Generation API

Generate a secure token for bank-hosted payment page.

**Endpoint:** `POST /api/generateToken`

**Request:**
```json
{
  "merchantId": "M123456",
  "tranportalId": "TRAN123",
  "tranportalPassword": "{encrypted}",
  "amount": "100.00",
  "currency": "SAR",
  "orderId": "ORD-2024-001",
  "orderDescription": "Purchase of product XYZ",
  "returnUrl": "https://merchant.com/payment/return",
  "customerName": "Ahmad Ali",
  "customerEmail": "ahmad@example.com",
  "customerPhone": "+966501234567",
  "billingAddress": {
    "street": "King Fahd Road",
    "city": "Riyadh",
    "state": "Riyadh",
    "country": "SA",
    "postalCode": "11564"
  }
}
```

**Response:**
```json
{
  "responseCode": "000",
  "responseMessage": "Success",
  "token": "TKN_1234567890ABCDEF",
  "paymentUrl": "https://arb.gateway.com/checkout?token=TKN_1234567890ABCDEF",
  "expiryTime": "2024-02-08T16:00:00Z",
  "orderId": "ORD-2024-001"
}
```

**Error Response:**
```json
{
  "responseCode": "E001",
  "responseMessage": "Invalid merchant credentials",
  "errorDetails": "Merchant ID or password is incorrect"
}
```

### Transaction Inquiry API

Query the status and details of a transaction.

**Endpoint:** `GET /api/transaction/{transactionId}`

**Alternative:** `POST /api/inquireTransaction`

**Request:**
```json
{
  "merchantId": "M123456",
  "tranportalId": "TRAN123",
  "tranportalPassword": "{encrypted}",
  "transactionId": "TXN123456789",
  "orderId": "ORD-2024-001"
}
```

**Response (Success):**
```json
{
  "responseCode": "000",
  "transactionId": "TXN123456789",
  "orderId": "ORD-2024-001",
  "status": "SUCCESS",
  "amount": "100.00",
  "currency": "SAR",
  "authorizationCode": "123456",
  "rrn": "123456789012",
  "paymentMethod": "VISA",
  "cardNumber": "411111******1111",
  "cardBrand": "VISA",
  "transactionDate": "2024-02-08T15:30:00Z",
  "merchantReference": "REF-001",
  "responseDescription": "Approved"
}
```

**Response (Pending):**
```json
{
  "responseCode": "001",
  "transactionId": "TXN123456789",
  "orderId": "ORD-2024-001",
  "status": "PENDING",
  "amount": "100.00",
  "currency": "SAR",
  "transactionDate": "2024-02-08T15:30:00Z",
  "responseDescription": "Transaction pending"
}
```

**Response (Failed):**
```json
{
  "responseCode": "002",
  "transactionId": "TXN123456789",
  "orderId": "ORD-2024-001",
  "status": "FAILED",
  "amount": "100.00",
  "currency": "SAR",
  "errorCode": "51",
  "errorDescription": "Insufficient funds",
  "transactionDate": "2024-02-08T15:30:00Z"
}
```

### Refund API

Process full or partial refunds for successful transactions.

**Endpoint:** `POST /api/refund`

**Request:**
```json
{
  "merchantId": "M123456",
  "tranportalId": "TRAN123",
  "tranportalPassword": "{encrypted}",
  "originalTransactionId": "TXN123456789",
  "refundAmount": "50.00",
  "currency": "SAR",
  "refundReason": "Customer request",
  "refundReference": "REF-REFUND-001"
}
```

**Response:**
```json
{
  "responseCode": "000",
  "responseMessage": "Refund processed successfully",
  "refundTransactionId": "RTXN987654321",
  "originalTransactionId": "TXN123456789",
  "refundAmount": "50.00",
  "currency": "SAR",
  "refundDate": "2024-02-08T16:00:00Z",
  "status": "SUCCESS",
  "rrn": "987654321098"
}
```

**Partial Refund:**
- Multiple partial refunds allowed
- Total cannot exceed original amount
- Each refund generates unique transaction ID

**Refund Limitations:**
- Must be initiated within 180 days of original transaction
- Minimum refund amount: 1.00 SAR
- Original transaction must be successful
- Cannot refund already voided transactions

### Void API

Cancel an authorized transaction before settlement.

**Endpoint:** `POST /api/void`

**Request:**
```json
{
  "merchantId": "M123456",
  "tranportalId": "TRAN123",
  "tranportalPassword": "{encrypted}",
  "transactionId": "TXN123456789",
  "voidReason": "Duplicate transaction"
}
```

**Response:**
```json
{
  "responseCode": "000",
  "responseMessage": "Transaction voided successfully",
  "voidTransactionId": "VTXN456789123",
  "originalTransactionId": "TXN123456789",
  "voidDate": "2024-02-08T16:15:00Z",
  "status": "VOIDED"
}
```

**Void Limitations:**
- Can only void transactions on same day before settlement
- Cannot void refunded transactions
- Cannot void already captured transactions
- Settlement typically occurs at end of business day

### Capture API

Capture a previously authorized amount (for two-step transactions).

**Endpoint:** `POST /api/capture`

**Request:**
```json
{
  "merchantId": "M123456",
  "tranportalId": "TRAN123",
  "tranportalPassword": "{encrypted}",
  "authorizationId": "AUTH123456789",
  "captureAmount": "100.00",
  "currency": "SAR",
  "finalCapture": true
}
```

**Response:**
```json
{
  "responseCode": "000",
  "responseMessage": "Amount captured successfully",
  "captureTransactionId": "CTXN789123456",
  "authorizationId": "AUTH123456789",
  "capturedAmount": "100.00",
  "currency": "SAR",
  "captureDate": "2024-02-08T16:30:00Z",
  "status": "CAPTURED"
}
```

**Capture Options:**
- **Full Capture**: Capture entire authorized amount
- **Partial Capture**: Capture less than authorized amount
- **Multiple Captures**: Multiple partial captures up to authorized amount
- **Final Capture**: Marks authorization as complete

### Direct Payment API (Merchant Hosted)

Process direct payment with encrypted card data.

**Endpoint:** `POST /api/rest/version/XX/merchant/{merchantId}/order/{orderId}/transaction/{transactionId}`

**Request:**
```json
{
  "apiOperation": "PAY",
  "order": {
    "amount": "100.00",
    "currency": "SAR",
    "description": "Product purchase"
  },
  "sourceOfFunds": {
    "type": "CARD",
    "provided": {
      "card": {
        "number": "{encrypted_card_number}",
        "securityCode": "{encrypted_cvv}",
        "expiry": {
          "month": "12",
          "year": "25"
        }
      }
    }
  },
  "customer": {
    "email": "customer@example.com",
    "firstName": "Ahmad",
    "lastName": "Ali",
    "phone": "+966501234567"
  },
  "billing": {
    "address": {
      "street": "King Fahd Road",
      "city": "Riyadh",
      "stateProvince": "Riyadh",
      "postcodeZip": "11564",
      "country": "SAR"
    }
  },
  "transaction": {
    "reference": "REF-001"
  }
}
```

**Response:**
```json
{
  "gatewayRecommendation": "PROCEED",
  "merchant": "M123456",
  "order": {
    "amount": "100.00",
    "currency": "SAR",
    "id": "ORD-2024-001",
    "status": "CAPTURED"
  },
  "response": {
    "gatewayCode": "APPROVED"
  },
  "result": "SUCCESS",
  "sourceOfFunds": {
    "provided": {
      "card": {
        "brand": "VISA",
        "fundingMethod": "CREDIT",
        "number": "411111xxxxxx1111",
        "scheme": "VISA"
      }
    },
    "type": "CARD"
  },
  "timeOfRecord": "2024-02-08T15:30:00.000Z",
  "transaction": {
    "authorizationCode": "123456",
    "id": "TXN123456789",
    "receipt": "123456789012",
    "type": "PAYMENT"
  },
  "version": "XX"
}
```

### Response Codes

| Code | Description | Action |
|------|-------------|--------|
| 000 | Success | Transaction completed successfully |
| 001 | Pending | Transaction is being processed |
| 002 | Failed | Transaction failed, check errorCode |
| E001 | Authentication Error | Check merchant credentials |
| E002 | Invalid Request | Verify request parameters |
| E003 | Transaction Not Found | Verify transaction ID |
| E004 | Refund Not Allowed | Check refund eligibility |
| E005 | Void Not Allowed | Transaction already settled |

## Security & Encryption

### AES Encryption Standards

ARB Payment Gateway uses AES (Advanced Encryption Standard) encryption for sensitive data transmission.

**Encryption Specifications:**
- **Algorithm**: AES (Advanced Encryption Standard)
- **Mode**: CBC (Cipher Block Chaining)
- **Padding**: PKCS5Padding
- **Key Size**: 256-bit (provided as Resource Key)
- **IV (Initialization Vector)**: "PGKEYENCDECIVSPC" (16 bytes)
- **Encoding**: Base64 for encrypted output

### Encryption Implementation

#### Java Example

```java
import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class ARBEncryption {
    
    private static final String ALGORITHM = "AES/CBC/PKCS5Padding";
    private static final String IV_STRING = "PGKEYENCDECIVSPC";
    
    public static String encrypt(String data, String resourceKey) throws Exception {
        // Prepare key
        SecretKeySpec secretKey = new SecretKeySpec(
            resourceKey.getBytes("UTF-8"), "AES"
        );
        
        // Prepare IV
        IvParameterSpec iv = new IvParameterSpec(
            IV_STRING.getBytes("UTF-8")
        );
        
        // Initialize cipher
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, iv);
        
        // Encrypt data
        byte[] encryptedBytes = cipher.doFinal(data.getBytes("UTF-8"));
        
        // Return Base64 encoded result
        return Base64.getEncoder().encodeToString(encryptedBytes);
    }
    
    public static String decrypt(String encryptedData, String resourceKey) throws Exception {
        // Prepare key
        SecretKeySpec secretKey = new SecretKeySpec(
            resourceKey.getBytes("UTF-8"), "AES"
        );
        
        // Prepare IV
        IvParameterSpec iv = new IvParameterSpec(
            IV_STRING.getBytes("UTF-8")
        );
        
        // Initialize cipher
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, secretKey, iv);
        
        // Decode Base64 and decrypt
        byte[] decodedBytes = Base64.getDecoder().decode(encryptedData);
        byte[] decryptedBytes = cipher.doFinal(decodedBytes);
        
        return new String(decryptedBytes, "UTF-8");
    }
    
    // Example usage
    public static void main(String[] args) {
        try {
            String resourceKey = "YOUR_32_CHAR_RESOURCE_KEY_HERE!";
            String cardNumber = "4111111111111111";
            
            // Encrypt
            String encrypted = encrypt(cardNumber, resourceKey);
            System.out.println("Encrypted: " + encrypted);
            
            // Decrypt
            String decrypted = decrypt(encrypted, resourceKey);
            System.out.println("Decrypted: " + decrypted);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### Node.js Example

```javascript
const crypto = require('crypto');

class ARBEncryption {
    constructor(resourceKey) {
        this.algorithm = 'aes-256-cbc';
        this.key = Buffer.from(resourceKey, 'utf8');
        this.iv = Buffer.from('PGKEYENCDECIVSPC', 'utf8');
    }
    
    encrypt(data) {
        try {
            const cipher = crypto.createCipheriv(
                this.algorithm,
                this.key,
                this.iv
            );
            
            let encrypted = cipher.update(data, 'utf8', 'base64');
            encrypted += cipher.final('base64');
            
            return encrypted;
        } catch (error) {
            console.error('Encryption error:', error);
            throw error;
        }
    }
    
    decrypt(encryptedData) {
        try {
            const decipher = crypto.createDecipheriv(
                this.algorithm,
                this.key,
                this.iv
            );
            
            let decrypted = decipher.update(encryptedData, 'base64', 'utf8');
            decrypted += decipher.final('utf8');
            
            return decrypted;
        } catch (error) {
            console.error('Decryption error:', error);
            throw error;
        }
    }
}

// Example usage
const resourceKey = 'YOUR_32_CHAR_RESOURCE_KEY_HERE!';
const encryption = new ARBEncryption(resourceKey);

// Encrypt card number
const cardNumber = '4111111111111111';
const encrypted = encryption.encrypt(cardNumber);
console.log('Encrypted:', encrypted);

// Decrypt
const decrypted = encryption.decrypt(encrypted);
console.log('Decrypted:', decrypted);

// Encrypt complete card data
const cardData = JSON.stringify({
    cardNumber: '4111111111111111',
    cvv: '123',
    expiryMonth: '12',
    expiryYear: '2025'
});

const encryptedCard = encryption.encrypt(cardData);
console.log('Encrypted Card Data:', encryptedCard);

module.exports = ARBEncryption;
```

#### Python Example

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import base64

class ARBEncryption:
    
    def __init__(self, resource_key):
        """
        Initialize encryption with resource key
        
        Args:
            resource_key (str): 32-character resource key from ARB
        """
        self.algorithm = AES.MODE_CBC
        self.key = resource_key.encode('utf-8')
        self.iv = b'PGKEYENCDECIVSPC'
        self.block_size = AES.block_size
    
    def encrypt(self, data):
        """
        Encrypt data using AES-256-CBC
        
        Args:
            data (str): Data to encrypt
            
        Returns:
            str: Base64 encoded encrypted data
        """
        try:
            # Create cipher
            cipher = AES.new(self.key, self.algorithm, self.iv)
            
            # Pad data to block size
            padded_data = pad(data.encode('utf-8'), self.block_size)
            
            # Encrypt
            encrypted_bytes = cipher.encrypt(padded_data)
            
            # Return Base64 encoded
            return base64.b64encode(encrypted_bytes).decode('utf-8')
            
        except Exception as e:
            print(f"Encryption error: {e}")
            raise
    
    def decrypt(self, encrypted_data):
        """
        Decrypt data using AES-256-CBC
        
        Args:
            encrypted_data (str): Base64 encoded encrypted data
            
        Returns:
            str: Decrypted data
        """
        try:
            # Decode Base64
            encrypted_bytes = base64.b64decode(encrypted_data)
            
            # Create cipher
            cipher = AES.new(self.key, self.algorithm, self.iv)
            
            # Decrypt
            decrypted_padded = cipher.decrypt(encrypted_bytes)
            
            # Unpad
            decrypted = unpad(decrypted_padded, self.block_size)
            
            return decrypted.decode('utf-8')
            
        except Exception as e:
            print(f"Decryption error: {e}")
            raise

# Example usage
if __name__ == "__main__":
    resource_key = "YOUR_32_CHAR_RESOURCE_KEY_HERE!"
    encryption = ARBEncryption(resource_key)
    
    # Encrypt card number
    card_number = "4111111111111111"
    encrypted = encryption.encrypt(card_number)
    print(f"Encrypted: {encrypted}")
    
    # Decrypt
    decrypted = encryption.decrypt(encrypted)
    print(f"Decrypted: {decrypted}")
    
    # Encrypt card data object
    import json
    card_data = json.dumps({
        "cardNumber": "4111111111111111",
        "cvv": "123",
        "expiryMonth": "12",
        "expiryYear": "2025"
    })
    
    encrypted_card = encryption.encrypt(card_data)
    print(f"Encrypted Card Data: {encrypted_card}")
    
    # Decrypt card data
    decrypted_card = encryption.decrypt(encrypted_card)
    print(f"Decrypted Card Data: {decrypted_card}")
```

### Encryption Best Practices

1. **Never Log Sensitive Data**
   ```javascript
   // DON'T
   console.log('Card number:', cardNumber);
   console.log('CVV:', cvv);
   
   // DO
   console.log('Card encrypted successfully');
   console.log('Transaction ID:', transactionId);
   ```

2. **Secure Key Storage**
   ```javascript
   // Store resource key in environment variables
   const resourceKey = process.env.ARB_RESOURCE_KEY;
   
   // Never hardcode in source code
   // const resourceKey = "MY_ACTUAL_KEY_123"; // DON'T DO THIS
   ```

3. **Validate Before Encryption**
   ```javascript
   function validateAndEncrypt(cardNumber, cvv, encryption) {
     // Validate card number (Luhn algorithm)
     if (!isValidCardNumber(cardNumber)) {
       throw new Error('Invalid card number');
     }
     
     // Validate CVV
     if (!/^\d{3,4}$/.test(cvv)) {
       throw new Error('Invalid CVV');
     }
     
     // Encrypt
     return {
       cardNumber: encryption.encrypt(cardNumber),
       cvv: encryption.encrypt(cvv)
     };
   }
   ```

4. **Handle Encryption Errors**
   ```javascript
   try {
     const encrypted = encryption.encrypt(sensitiveData);
     // Proceed with API call
   } catch (error) {
     console.error('Encryption failed');
     // Don't expose encryption details to user
     showUserError('Payment processing error. Please try again.');
   }
   ```

### SSL/TLS Requirements

All API communications must use HTTPS with TLS 1.2 or higher:

```javascript
// Node.js - Enforce TLS 1.2+
const https = require('https');
const options = {
  hostname: 'arb.gateway.mastercard.com',
  port: 443,
  path: '/api/rest/version/XX/merchant/...',
  method: 'POST',
  secureProtocol: 'TLSv1_2_method',
  rejectUnauthorized: true
};
```

### Password Encryption for Tranportal

The Tranportal password must also be encrypted using the same AES method:

```javascript
const encryption = new ARBEncryption(resourceKey);
const encryptedPassword = encryption.encrypt(tranportalPassword);

// Use in API request
const request = {
  tranportalId: 'TRAN123',
  tranportalPassword: encryptedPassword,
  // ... other parameters
};
```

### Data Security Guidelines

1. **PCI DSS Compliance**
   - Never store CVV/CVC codes
   - Mask card numbers (show only last 4 digits)
   - Encrypt all card data in transit
   - Use tokenization for stored cards
   - Implement access controls

2. **Network Security**
   - Use VPN for API access (if required)
   - Whitelist IP addresses in Tranportal
   - Implement rate limiting
   - Monitor for suspicious activity
   - Use firewall protection

3. **Application Security**
   - Validate all inputs
   - Sanitize user data
   - Implement CSRF protection
   - Use secure session management
   - Regular security audits

4. **Data Retention**
   - Delete expired tokens
   - Purge transaction logs per policy
   - Securely delete sensitive data
   - Maintain audit trails
   - Follow data privacy regulations

## Payment Methods

### Credit and Debit Cards

ARB Payment Gateway supports major international and local card schemes.

#### Supported Card Types

**International Cards:**
- **Visa**: Credit and debit cards
- **Mastercard**: Credit and debit cards

**Local Cards (Saudi Arabia):**
- **MADA**: Saudi domestic debit card scheme
  - Issued by all Saudi banks
  - Requires 3D Secure authentication
  - SAR transactions only
  - Highest success rate in Saudi Arabia

#### Card Validation

**Card Number Validation (Luhn Algorithm):**
```javascript
function validateCardNumber(cardNumber) {
  // Remove spaces and hyphens
  const cleaned = cardNumber.replace(/[\s-]/g, '');
  
  // Check if only digits
  if (!/^\d+$/.test(cleaned)) {
    return false;
  }
  
  // Check length (13-19 digits)
  if (cleaned.length < 13 || cleaned.length > 19) {
    return false;
  }
  
  // Luhn algorithm
  let sum = 0;
  let isEven = false;
  
  for (let i = cleaned.length - 1; i >= 0; i--) {
    let digit = parseInt(cleaned[i]);
    
    if (isEven) {
      digit *= 2;
      if (digit > 9) {
        digit -= 9;
      }
    }
    
    sum += digit;
    isEven = !isEven;
  }
  
  return sum % 10 === 0;
}
```

**Card Type Detection:**
```javascript
function detectCardType(cardNumber) {
  const cleaned = cardNumber.replace(/[\s-]/g, '');
  
  // Visa: starts with 4
  if (/^4/.test(cleaned)) {
    return 'VISA';
  }
  
  // Mastercard: starts with 51-55 or 2221-2720
  if (/^5[1-5]/.test(cleaned) || /^2(22[1-9]|2[3-9]\d|[3-6]\d\d|7[01]\d|720)/.test(cleaned)) {
    return 'MASTERCARD';
  }
  
  // MADA: various BINs
  const madaBins = [
    '588845', '440647', '440795', '446404', '457997',
    '968201', '968203', '968205', '968207', '968208',
    '508160', '588848', '588850', '589005', '604906',
    '968211'
  ];
  
  for (const bin of madaBins) {
    if (cleaned.startsWith(bin)) {
      return 'MADA';
    }
  }
  
  return 'UNKNOWN';
}
```

#### Card Processing Features

1. **3D Secure (3DS)**
   - Mandatory for MADA
   - Recommended for international cards
   - Reduces fraud and chargebacks
   - Supports 3DS 2.0 protocol

2. **CVV Verification**
   - Required for card-not-present transactions
   - 3 digits (Visa, Mastercard, MADA)
   - 4 digits (American Express)

3. **Address Verification (AVS)**
   - Available for international cards
   - Compares billing address with issuer records
   - Helps prevent fraud

### Apple Pay

Seamless mobile payment integration for iOS devices.

#### Prerequisites

1. **Apple Developer Account**
2. **Merchant Identifier** from Apple
3. **Payment Processing Certificate**
4. **Domain Verification**

#### Integration Steps

**1. Configure Merchant Identifier**
```javascript
const applePayConfig = {
  merchantIdentifier: 'merchant.com.yourcompany.app',
  merchantName: 'Your Company Name',
  countryCode: 'SA',
  currencyCode: 'SAR'
};
```

**2. Check Apple Pay Availability**
```javascript
if (window.ApplePaySession) {
  if (ApplePaySession.canMakePayments()) {
    // Show Apple Pay button
    displayApplePayButton();
  }
}
```

**3. Create Payment Request**
```javascript
const paymentRequest = {
  countryCode: 'SA',
  currencyCode: 'SAR',
  supportedNetworks: ['visa', 'masterCard', 'mada'],
  merchantCapabilities: ['supports3DS'],
  total: {
    label: 'Your Company Name',
    amount: '100.00',
    type: 'final'
  },
  lineItems: [
    {
      label: 'Product 1',
      amount: '80.00'
    },
    {
      label: 'Shipping',
      amount: '20.00'
    }
  ]
};
```

**4. Process Apple Pay Payment**
```javascript
async function processApplePayPayment(paymentToken) {
  const response = await fetch('/api/apple-pay/process', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      merchantId: 'M123456',
      amount: '100.00',
      currency: 'SAR',
      applePayToken: {
        paymentData: paymentToken.paymentData,
        paymentMethod: paymentToken.paymentMethod,
        transactionIdentifier: paymentToken.transactionIdentifier
      }
    })
  });
  
  return await response.json();
}
```

#### Apple Pay Benefits

- **Fast Checkout**: Single-touch payment
- **Secure**: Tokenized transactions
- **No Card Entry**: Uses stored cards
- **Biometric Auth**: Face ID / Touch ID
- **High Conversion**: Reduced friction

### URPay

Saudi Arabia's instant payment system.

#### Features

- Real-time bank transfers
- 24/7 availability
- Direct bank-to-bank payments
- QR code support
- Mobile app integration

#### Integration

**1. Generate URPay Payment Request**
```javascript
const urpayRequest = {
  merchantId: 'M123456',
  amount: '100.00',
  currency: 'SAR',
  paymentMethod: 'URPAY',
  orderId: 'ORD-2024-001',
  customerMobile: '+966501234567',
  description: 'Product purchase'
};
```

**2. Display QR Code**
- Generate QR code from payment request
- Customer scans with URPay app
- Instant payment confirmation

**3. Handle Callback**
```javascript
app.post('/urpay/callback', (req, res) => {
  const { transactionId, status, amount } = req.body;
  
  if (status === 'SUCCESS') {
    // Update order status
    updateOrderStatus(transactionId, 'PAID');
  }
  
  res.status(200).send('OK');
});
```

### SADAD

Saudi Arabia's bill payment system.

#### Features

- Bill payment and invoicing
- Integrated with all Saudi banks
- Customer pays through ATM or online banking
- Batch payment processing

#### SADAD Integration

**1. Generate SADAD Bill**
```javascript
const sadadBill = {
  merchantId: 'M123456',
  billNumber: 'BILL-2024-001',
  amount: '100.00',
  currency: 'SAR',
  dueDate: '2024-03-01',
  customerName: 'Ahmad Ali',
  customerAccount: 'ACC123456',
  description: 'Service payment'
};
```

**2. Customer Payment Flow**
- Customer receives SADAD bill number
- Logs into bank account or visits ATM
- Enters SADAD bill number
- Confirms payment
- Merchant receives payment notification

**3. Payment Reconciliation**
```javascript
// Query SADAD payments
const payments = await querySadadPayments({
  merchantId: 'M123456',
  fromDate: '2024-02-01',
  toDate: '2024-02-08'
});

// Process each payment
payments.forEach(payment => {
  if (payment.status === 'PAID') {
    updateInvoiceStatus(payment.billNumber, 'PAID');
  }
});
```

### Credit Card Installments

Allow customers to pay in installments.

#### Supported Banks

- Al Rajhi Bank
- National Commercial Bank (NCB)
- Riyad Bank
- Samba Bank
- Saudi Investment Bank
- Others (check with ARB)

#### Installment Options

- **3 months**: 0% interest (typical)
- **6 months**: 0% or low interest
- **12 months**: Low interest rate
- **24 months**: Standard interest rate

#### Implementation

**1. Check Installment Eligibility**
```javascript
const eligibility = await checkInstallmentEligibility({
  cardNumber: encryptedCardNumber,
  amount: '3000.00',
  currency: 'SAR'
});

// Response
{
  "eligible": true,
  "plans": [
    {
      "months": 3,
      "monthlyAmount": "1000.00",
      "interestRate": "0%",
      "totalAmount": "3000.00"
    },
    {
      "months": 6,
      "monthlyAmount": "500.00",
      "interestRate": "0%",
      "totalAmount": "3000.00"
    },
    {
      "months": 12,
      "monthlyAmount": "258.33",
      "interestRate": "3.5%",
      "totalAmount": "3100.00"
    }
  ]
}
```

**2. Process Installment Payment**
```javascript
const payment = await processPayment({
  merchantId: 'M123456',
  amount: '3000.00',
  currency: 'SAR',
  cardData: encryptedCardData,
  installment: {
    enabled: true,
    plan: 6,  // 6 months
    monthlyAmount: '500.00'
  }
});
```

**3. Display Installment Options to Customer**
```html
<div class="installment-options">
  <h3>Pay in Installments</h3>
  
  <label>
    <input type="radio" name="installment" value="3">
    3 months × 1,000.00 SAR (0% interest)
  </label>
  
  <label>
    <input type="radio" name="installment" value="6">
    6 months × 500.00 SAR (0% interest)
  </label>
  
  <label>
    <input type="radio" name="installment" value="12">
    12 months × 258.33 SAR (3.5% interest)
  </label>
</div>
```

### Split Payments

Split a transaction across multiple payment methods.

#### Use Cases

- Part payment with gift card, remainder with card
- Split between multiple cards
- Partial wallet payment with card balance
- Corporate and personal card split

#### Implementation

**1. Create Split Payment Request**
```javascript
const splitPayment = {
  merchantId: 'M123456',
  orderId: 'ORD-2024-001',
  totalAmount: '500.00',
  currency: 'SAR',
  splits: [
    {
      method: 'GIFT_CARD',
      amount: '100.00',
      giftCardNumber: 'GIFT123456'
    },
    {
      method: 'CARD',
      amount: '400.00',
      cardData: encryptedCardData
    }
  ]
};
```

**2. Process Each Split**
```javascript
async function processSplitPayment(splitPayment) {
  const results = [];
  
  for (const split of splitPayment.splits) {
    try {
      const result = await processPaymentSplit(split);
      results.push(result);
      
      if (result.status !== 'SUCCESS') {
        // Reverse previous successful splits
        await reverseSuccessfulSplits(results);
        throw new Error('Split payment failed');
      }
    } catch (error) {
      await reverseSuccessfulSplits(results);
      throw error;
    }
  }
  
  return {
    status: 'SUCCESS',
    transactionId: generateTransactionId(),
    splits: results
  };
}
```

### Payment Method Selection

**Detect Available Methods:**
```javascript
function getAvailablePaymentMethods(amount, currency, country) {
  const methods = ['CARD'];
  
  // Check Apple Pay
  if (window.ApplePaySession && ApplePaySession.canMakePayments()) {
    methods.push('APPLE_PAY');
  }
  
  // Check MADA (Saudi Arabia only)
  if (country === 'SA') {
    methods.push('MADA', 'SADAD', 'URPAY');
  }
  
  // Check installments (minimum amount)
  if (amount >= 1000 && currency === 'SAR') {
    methods.push('INSTALLMENTS');
  }
  
  return methods;
}
```

## Testing

### Sandbox Environment

Always test your integration in the sandbox environment before going live.

**Sandbox Endpoints:**
```
Gateway: https://test.arb.gateway.mastercard.com
Tranportal: https://sandbox.tranportal.alrajhibank.com.sa
```

**Sandbox Credentials:**
- Request sandbox credentials from ARB merchant services
- Separate credentials from production
- No real money transactions
- Full feature parity with production

### Test Cards

ARB provides test card numbers for various scenarios.

#### MADA Test Cards

**Successful Transaction:**
```
Card Number: 5297410000000000
Expiry: Any future date (e.g., 12/25)
CVV: 123
Name: TEST CARDHOLDER
3D Secure OTP: 123456
```

**Successful Transaction (Alternative):**
```
Card Number: 5043310000000000
Expiry: 12/25
CVV: 123
Name: TEST USER
3D Secure OTP: 123456
```

**Insufficient Funds:**
```
Card Number: 5297410000000018
Expiry: 12/25
CVV: 123
Expected Result: Declined - Insufficient funds (Code 51)
```

**Invalid Card:**
```
Card Number: 5297410000000026
Expiry: 12/25
CVV: 123
Expected Result: Declined - Invalid card (Code 14)
```

**Expired Card:**
```
Card Number: 5297410000000034
Expiry: 12/23 (past date)
CVV: 123
Expected Result: Declined - Expired card (Code 54)
```

#### Visa Test Cards

**Successful Transaction:**
```
Card Number: 4111111111111111
Expiry: 12/25
CVV: 123
Name: VISA TEST
3D Secure OTP: 1234 (if prompted)
```

**Successful Transaction (Alternative):**
```
Card Number: 4012000033330026
Expiry: 12/25
CVV: 123
Name: VISA HOLDER
```

**Insufficient Funds:**
```
Card Number: 4111111111111129
Expiry: 12/25
CVV: 123
Expected Result: Declined - Insufficient funds (Code 51)
```

**Invalid CVV:**
```
Card Number: 4111111111111111
Expiry: 12/25
CVV: 999
Expected Result: Declined - Invalid CVV (Code 55)
```

**3D Secure Authentication Failed:**
```
Card Number: 4012000033330026
Expiry: 12/25
CVV: 123
3D Secure: Cancel or enter wrong OTP
Expected Result: Authentication failed
```

#### Mastercard Test Cards

**Successful Transaction:**
```
Card Number: 5500000000000004
Expiry: 12/25
CVV: 123
Name: MASTERCARD TEST
```

**Successful Transaction (Alternative):**
```
Card Number: 5555555555554444
Expiry: 12/25
CVV: 123
Name: MC HOLDER
```

**Insufficient Funds:**
```
Card Number: 5500000000000012
Expiry: 12/25
CVV: 123
Expected Result: Declined - Insufficient funds (Code 51)
```

**Do Not Honor:**
```
Card Number: 5500000000000020
Expiry: 12/25
CVV: 123
Expected Result: Declined - Do not honor (Code 05)
```

**Lost Card:**
```
Card Number: 5500000000000038
Expiry: 12/25
CVV: 123
Expected Result: Declined - Lost card (Code 41)
```

### Test Scenarios

#### Scenario 1: Successful Payment

**Test Steps:**
1. Initiate payment with test card (e.g., 4111111111111111)
2. Enter valid expiry and CVV
3. Complete 3D Secure authentication with test OTP (1234 or 123456)
4. Verify transaction status is SUCCESS
5. Check authorization code is returned
6. Confirm amount is correct

**Expected Result:**
- Status: SUCCESS
- Response Code: 000 or 00
- Authorization Code: Present
- Transaction ID: Generated

#### Scenario 2: Insufficient Funds

**Test Steps:**
1. Initiate payment with insufficient funds test card
2. Complete payment form
3. Attempt transaction

**Expected Result:**
- Status: FAILED
- Response Code: 51
- Error Message: "Insufficient funds"
- No authorization code

#### Scenario 3: 3D Secure Failure

**Test Steps:**
1. Initiate payment with 3DS card
2. Enter card details
3. On 3DS authentication page, click Cancel or enter wrong OTP
4. Verify transaction fails

**Expected Result:**
- Status: FAILED
- Response Code: Authentication failed
- Transaction not authorized

#### Scenario 4: Expired Card

**Test Steps:**
1. Use test card with past expiry date
2. Attempt payment

**Expected Result:**
- Status: FAILED
- Response Code: 54
- Error Message: "Expired card"

#### Scenario 5: Refund Processing

**Test Steps:**
1. Complete successful test payment
2. Note transaction ID
3. Initiate refund request
4. Verify refund status

**Expected Result:**
- Refund Status: SUCCESS
- Refund Transaction ID: Generated
- Original transaction linked
- Amount refunded correctly

#### Scenario 6: Tokenization

**Test Steps:**
1. Complete payment with storeCard: true
2. Verify token is returned
3. Use token for subsequent payment
4. Confirm payment succeeds without re-entering card

**Expected Result:**
- Token generated on first payment
- Token can be reused
- Subsequent payment succeeds
- Card details not required again

#### Scenario 7: Apple Pay (if applicable)

**Test Steps:**
1. Use Safari on iOS device or Mac
2. Add test card to Apple Wallet
3. Initiate Apple Pay payment
4. Authenticate with Face ID/Touch ID
5. Complete payment

**Expected Result:**
- Apple Pay sheet displays correctly
- Payment processes successfully
- Token received from Apple
- Transaction completes

### Testing Checklist

**Pre-Integration:**
- [ ] Sandbox credentials obtained
- [ ] Test environment configured
- [ ] Encryption implementation tested
- [ ] Test cards documented

**Integration Testing:**
- [ ] Successful payment flow
- [ ] Failed payment scenarios
- [ ] 3D Secure authentication
- [ ] Refund processing
- [ ] Void transactions
- [ ] Token generation and usage
- [ ] Error handling
- [ ] Timeout scenarios

**Payment Methods:**
- [ ] MADA cards
- [ ] Visa cards
- [ ] Mastercard cards
- [ ] Apple Pay (if applicable)
- [ ] SADAD (if applicable)
- [ ] URPay (if applicable)

**Security Testing:**
- [ ] SSL/TLS verification
- [ ] Encryption/decryption
- [ ] API authentication
- [ ] Input validation
- [ ] XSS protection
- [ ] CSRF protection

**UI/UX Testing:**
- [ ] Payment form validation
- [ ] Loading states
- [ ] Error messages display
- [ ] Success confirmation
- [ ] Mobile responsiveness
- [ ] Browser compatibility

**Performance Testing:**
- [ ] API response times
- [ ] High volume transactions
- [ ] Concurrent requests
- [ ] Timeout handling
- [ ] Network interruptions

### Test Automation

**Example Test Suite (Node.js with Jest):**

```javascript
const ARBPaymentGateway = require('./arb-gateway');
const ARBEncryption = require('./arb-encryption');

describe('ARB Payment Gateway Integration', () => {
  let gateway;
  let encryption;
  
  beforeAll(() => {
    gateway = new ARBPaymentGateway({
      merchantId: process.env.TEST_MERCHANT_ID,
      tranportalId: process.env.TEST_TRANPORTAL_ID,
      tranportalPassword: process.env.TEST_TRANPORTAL_PASSWORD,
      environment: 'sandbox'
    });
    
    encryption = new ARBEncryption(process.env.TEST_RESOURCE_KEY);
  });
  
  test('Successful payment with Visa test card', async () => {
    const cardData = {
      cardNumber: '4111111111111111',
      cvv: '123',
      expiryMonth: '12',
      expiryYear: '2025'
    };
    
    const result = await gateway.processPayment({
      amount: '100.00',
      currency: 'SAR',
      cardData: cardData,
      orderId: `TEST-${Date.now()}`
    });
    
    expect(result.status).toBe('SUCCESS');
    expect(result.responseCode).toBe('000');
    expect(result.authorizationCode).toBeDefined();
    expect(result.transactionId).toBeDefined();
  });
  
  test('Declined payment - insufficient funds', async () => {
    const cardData = {
      cardNumber: '4111111111111129',
      cvv: '123',
      expiryMonth: '12',
      expiryYear: '2025'
    };
    
    const result = await gateway.processPayment({
      amount: '100.00',
      currency: 'SAR',
      cardData: cardData,
      orderId: `TEST-${Date.now()}`
    });
    
    expect(result.status).toBe('FAILED');
    expect(result.errorCode).toBe('51');
  });
  
  test('Refund successful transaction', async () => {
    // First, make successful payment
    const payment = await gateway.processPayment({
      amount: '100.00',
      currency: 'SAR',
      cardData: {
        cardNumber: '4111111111111111',
        cvv: '123',
        expiryMonth: '12',
        expiryYear: '2025'
      },
      orderId: `TEST-${Date.now()}`
    });
    
    // Then refund it
    const refund = await gateway.refund({
      transactionId: payment.transactionId,
      amount: '50.00',
      reason: 'Test refund'
    });
    
    expect(refund.status).toBe('SUCCESS');
    expect(refund.refundAmount).toBe('50.00');
  });
  
  test('Encryption and decryption', () => {
    const original = '4111111111111111';
    const encrypted = encryption.encrypt(original);
    const decrypted = encryption.decrypt(encrypted);
    
    expect(encrypted).not.toBe(original);
    expect(decrypted).toBe(original);
  });
});
```

### Debugging Tips

**1. Enable Verbose Logging:**
```javascript
const gateway = new ARBPaymentGateway({
  merchantId: 'M123456',
  environment: 'sandbox',
  debug: true  // Enable debug mode
});
```

**2. Log API Requests/Responses:**
```javascript
// Log request
console.log('API Request:', {
  endpoint: endpoint,
  method: method,
  payload: sanitizeForLog(payload)  // Remove sensitive data
});

// Log response
console.log('API Response:', {
  status: response.status,
  data: response.data
});
```

**3. Common Issues:**

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| Authentication failed | Wrong credentials | Verify merchant ID and password |
| Encryption error | Wrong resource key | Check resource key from Tranportal |
| 3DS timeout | Slow network | Increase timeout settings |
| Invalid card | Using production card in sandbox | Use test cards only |
| Transaction not found | Wrong transaction ID | Verify transaction ID format |

**4. Network Debugging:**
```bash
# Test API connectivity
curl -v https://test.arb.gateway.mastercard.com/api/health

# Test with proper auth
curl -X GET "https://test.arb.gateway.mastercard.com/api/merchant/info" \
  -H "Authorization: Basic {BASE64_ENCODED_CREDENTIALS}" \
  -v
```

## Troubleshooting

### Common Error Codes

ARB Payment Gateway returns specific error codes for different scenarios.

#### Gateway Error Codes (E-series)

| Code | Description | Resolution |
|------|-------------|------------|
| E001 | Invalid Merchant Credentials | Verify Merchant ID and Tranportal password are correct. Ensure credentials are properly encrypted. |
| E002 | Invalid Request Format | Check JSON structure and required fields. Verify API version compatibility. |
| E003 | Transaction Not Found | Verify transaction ID is correct. Check if transaction exists in current environment (sandbox/production). |
| E004 | Refund Not Allowed | Ensure original transaction was successful. Verify refund amount doesn't exceed original amount. Check transaction age (< 180 days). |
| E005 | Void Not Allowed | Transaction already settled or void window expired. Use refund instead of void for settled transactions. |
| E006 | Invalid Amount | Check amount format (e.g., "100.00"). Verify currency matches merchant configuration. Ensure amount is within allowed limits. |
| E007 | Invalid Currency | Use supported currency code (e.g., "SAR"). Verify merchant account supports requested currency. |
| E008 | Duplicate Transaction | Check if transaction was already processed. Implement idempotency with unique order IDs. |
| E009 | Encryption Error | Verify resource key is correct. Check encryption algorithm (AES-256-CBC). Ensure IV is "PGKEYENCDECIVSPC". |
| E010 | Invalid Card Data | Validate card number (Luhn check). Verify expiry date format and future date. Check CVV length (3-4 digits). |
| E011 | Merchant Account Suspended | Contact ARB merchant services. Verify account status in Tranportal. |
| E012 | Transaction Limit Exceeded | Check daily/monthly transaction limits. Contact ARB to increase limits if needed. |
| E013 | IP Not Whitelisted | Add your server IP to whitelist in Tranportal. Verify firewall settings. |
| E014 | Invalid Callback URL | Ensure return URL uses HTTPS. Verify URL is accessible. Check URL format. |
| E015 | Session Timeout | Payment session expired (typically 15-30 minutes). Initiate new payment. |

#### Authorization Response Codes

| Code | Description | Customer Message | Merchant Action |
|------|-------------|------------------|-----------------|
| 00 | Approved | Payment successful | Complete order |
| 01 | Refer to card issuer | Please contact your bank | Suggest alternative payment |
| 03 | Invalid merchant | Technical error occurred | Check merchant configuration |
| 04 | Pick up card | Payment declined | Contact card issuer |
| 05 | Do not honor | Payment declined by bank | Suggest alternative payment |
| 12 | Invalid transaction | Transaction cannot be processed | Check transaction parameters |
| 13 | Invalid amount | Invalid payment amount | Verify amount format |
| 14 | Invalid card number | Invalid card number | Re-enter card details |
| 30 | Format error | Technical error occurred | Check request format |
| 41 | Lost card | Card reported as lost | Cannot process, suggest alternative |
| 43 | Stolen card | Card reported as stolen | Cannot process, suggest alternative |
| 51 | Insufficient funds | Insufficient funds in account | Suggest alternative payment |
| 54 | Expired card | Card has expired | Update card details |
| 55 | Incorrect PIN/CVV | Incorrect security code | Re-enter CVV |
| 57 | Transaction not permitted | Card cannot be used for this transaction | Use different card |
| 58 | Transaction not allowed | Transaction type not allowed | Contact card issuer |
| 61 | Exceeds withdrawal limit | Transaction exceeds card limit | Reduce amount or use different card |
| 62 | Restricted card | Card is restricted | Use different card |
| 63 | Security violation | Security check failed | Contact card issuer |
| 65 | Exceeds frequency limit | Too many transactions | Wait and retry or use different card |
| 75 | PIN tries exceeded | Too many incorrect attempts | Card temporarily blocked |
| 76 | Invalid/nonexistent account | Account issue | Use different card |
| 91 | Issuer unavailable | Bank system unavailable | Retry later |
| 92 | Unable to route | Cannot reach card issuer | Retry or use different card |
| 93 | Cannot complete | Transaction cannot be completed | Contact card issuer |
| 94 | Duplicate transmission | Duplicate transaction detected | Check if original succeeded |
| 96 | System malfunction | Technical error | Retry transaction |

### 3D Secure Issues

#### Issue: 3DS Authentication Window Not Appearing

**Causes:**
- Pop-up blocker enabled
- JavaScript errors
- Invalid redirect URL
- Browser compatibility

**Solutions:**
```javascript
// Check pop-up blocker
function check3DSSupport() {
  const popup = window.open('', '_blank', 'width=1,height=1');
  if (popup) {
    popup.close();
    return true;
  } else {
    alert('Please enable pop-ups for payment authentication');
    return false;
  }
}

// Use iframe instead of popup
<iframe id="3ds-frame" src="{3DS_URL}" width="100%" height="600px"></iframe>
```

#### Issue: 3DS Authentication Timeout

**Causes:**
- Customer took too long to complete auth
- Network issues
- Bank system slow response

**Solutions:**
- Increase timeout window (if configurable)
- Display clear timer to customer
- Allow retry without re-entering card details
- Implement proper error handling

```javascript
const timeout = setTimeout(() => {
  show3DSTimeoutMessage();
  enable RetryButton();
}, 300000); // 5 minutes

// Clear timeout on success
on3DSSuccess(() => {
  clearTimeout(timeout);
});
```

### Encryption Issues

#### Issue: Encryption Produces Different Results

**Problem:** Encrypted output differs from expected

**Cause:** Inconsistent encoding or padding

**Solution:**
```javascript
// Ensure consistent encoding
function encryptConsistent(data, key) {
  // Always convert to UTF-8
  const dataBuffer = Buffer.from(data, 'utf8');
  const keyBuffer = Buffer.from(key, 'utf8');
  const ivBuffer = Buffer.from('PGKEYENCDECIVSPC', 'utf8');
  
  const cipher = crypto.createCipheriv('aes-256-cbc', keyBuffer, ivBuffer);
  
  let encrypted = cipher.update(dataBuffer);
  encrypted = Buffer.concat([encrypted, cipher.final()]);
  
  // Always return base64
  return encrypted.toString('base64');
}
```

#### Issue: Decryption Fails on Server Side

**Causes:**
- Wrong resource key
- Incorrect IV
- Encoding mismatch
- Padding issue

**Solutions:**
- Verify resource key matches Tranportal
- Confirm IV is exactly "PGKEYENCDECIVSPC"
- Use UTF-8 encoding consistently
- Ensure PKCS5Padding is used

### Integration Issues

#### Issue: CORS Errors in Browser

**Problem:** Cross-origin requests blocked

**Solution:**
```javascript
// Don't make direct API calls from browser
// Use server-side proxy

// Client-side
fetch('/api/payment/process', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(paymentData)
});

// Server-side proxy
app.post('/api/payment/process', async (req, res) => {
  const result = await arbGateway.processPayment(req.body);
  res.json(result);
});
```

#### Issue: Webhook Not Received

**Causes:**
- Webhook URL not configured
- HTTPS required but using HTTP
- Firewall blocking ARB IPs
- Server not responding with 200

**Solutions:**
1. Configure webhook URL in Tranportal
2. Use HTTPS endpoints only
3. Whitelist ARB IP ranges
4. Return 200 status quickly

```javascript
app.post('/webhook/arb', express.json(), async (req, res) => {
  // Respond immediately
  res.status(200).send('OK');
  
  // Process webhook asynchronously
  processWebhookAsync(req.body).catch(error => {
    console.error('Webhook processing error:', error);
  });
});
```

#### Issue: Transaction Status Query Returns Null

**Causes:**
- Transaction ID incorrect
- Wrong environment (sandbox vs production)
- Transaction not yet processed
- Network timeout

**Solutions:**
```javascript
async function queryTransactionWithRetry(transactionId, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const result = await gateway.queryTransaction(transactionId);
      
      if (result) {
        return result;
      }
      
      // Wait before retry
      await sleep(2000 * (i + 1)); // Exponential backoff
    } catch (error) {
      if (i === maxRetries - 1) throw error;
    }
  }
  
  throw new Error('Transaction not found after retries');
}
```

### Performance Issues

#### Issue: Slow API Response Times

**Causes:**
- Network latency
- Inefficient implementation
- Not using connection pooling
- Too many synchronous calls

**Solutions:**
```javascript
// Use connection pooling
const https = require('https');
const agent = new https.Agent({
  keepAlive: true,
  maxSockets: 50
});

// Batch operations when possible
async function processMultipleTransactions(transactions) {
  // Process in parallel with limit
  const limit = 10;
  const results = [];
  
  for (let i = 0; i < transactions.length; i += limit) {
    const batch = transactions.slice(i, i + limit);
    const batchResults = await Promise.all(
      batch.map(txn => processTransaction(txn))
    );
    results.push(...batchResults);
  }
  
  return results;
}
```

### Debugging Checklist

**When Transaction Fails:**
- [ ] Check response code and error message
- [ ] Verify merchant credentials
- [ ] Confirm encryption is working correctly
- [ ] Validate card details (Luhn check)
- [ ] Check environment (sandbox vs production)
- [ ] Review API request/response logs
- [ ] Verify network connectivity
- [ ] Check transaction amount and currency
- [ ] Ensure 3DS authentication completed
- [ ] Verify order ID is unique

**When Refund Fails:**
- [ ] Confirm original transaction was successful
- [ ] Check refund amount ≤ original amount
- [ ] Verify transaction is within refund window (180 days)
- [ ] Ensure transaction not already refunded
- [ ] Check merchant refund permissions
- [ ] Verify refund API parameters

**When 3DS Fails:**
- [ ] Check if pop-ups are blocked
- [ ] Verify redirect URLs are correct
- [ ] Ensure JavaScript is enabled
- [ ] Test in different browsers
- [ ] Check network connectivity
- [ ] Verify 3DS is enabled for card
- [ ] Check customer completed authentication

### Support and Resources

#### Contact Information

**ARB Merchant Services:**
- Email: merchantservices@alrajhibank.com.sa
- Phone: +966-11-2116000
- Support Hours: Sunday-Thursday, 8:00 AM - 5:00 PM (GMT+3)

**Technical Support:**
- Email: techsupport@alrajhibank.com.sa
- Portal: https://tranportal.alrajhibank.com.sa/support
- Emergency: 24/7 for production issues

#### Useful Resources

1. **Tranportal Dashboard**
   - View transaction reports
   - Configure merchant settings
   - Download statements
   - Manage API credentials

2. **API Documentation**
   - Request from ARB merchant services
   - Check for API updates
   - Review integration guides

3. **Status Page**
   - Monitor system availability
   - Check for maintenance windows
   - View incident history

#### Best Practices for Support Requests

When contacting support, include:

1. **Merchant Information**
   - Merchant ID
   - Environment (sandbox/production)
   - Integration type

2. **Transaction Details**
   - Transaction ID
   - Order ID
   - Transaction date/time
   - Amount and currency

3. **Error Information**
   - Error code
   - Error message
   - Request/response logs (sanitized)
   - Steps to reproduce

4. **Technical Details**
   - API version
   - Programming language/framework
   - Encryption method
   - Browser/device (if applicable)

**Example Support Request:**
```
Subject: Transaction Query Failure - E003 Error

Merchant ID: M123456
Environment: Production
Integration: Bank Hosted

Issue: Transaction inquiry returning E003 error

Transaction Details:
- Transaction ID: TXN20240208123456
- Order ID: ORD-2024-001
- Date: 2024-02-08 15:30:00 GMT+3
- Amount: 100.00 SAR

Error Details:
- Error Code: E003
- Error Message: "Transaction Not Found"
- Request Timestamp: 2024-02-08 15:35:00 GMT+3

Additional Info:
- Transaction was successful according to customer
- Customer received payment confirmation
- Query API called 5 minutes after payment
- Using API version XX

Please advise how to retrieve transaction details.
```

---

## Appendix

### Glossary

- **3D Secure (3DS)**: Authentication protocol for online card transactions
- **AES**: Advanced Encryption Standard
- **Authorization**: Approval to charge a card
- **AVS**: Address Verification Service
- **BIN**: Bank Identification Number (first 6 digits of card)
- **Capture**: Collecting funds from an authorized transaction
- **Chargeback**: Customer dispute of a transaction
- **CVV**: Card Verification Value (security code)
- **Issuer**: Bank that issued the card
- **MADA**: Saudi Arabia's local debit card scheme
- **Merchant**: Business accepting payments
- **PCI DSS**: Payment Card Industry Data Security Standard
- **Refund**: Returning funds to customer
- **RRN**: Retrieval Reference Number
- **Settlement**: Transfer of funds to merchant account
- **Token**: Secure representation of payment details
- **Tranportal**: ARB's merchant portal
- **Void**: Cancelling an authorized but not settled transaction

### Compliance Requirements

**PCI DSS:**
- Do not store CVV/CVC codes
- Encrypt cardholder data
- Maintain secure network
- Implement access controls
- Regular security testing

**Saudi Regulations:**
- Comply with SAMA regulations
- Follow local data privacy laws
- Maintain transaction records
- Report suspicious activities

### Change Log

**Version 2.0** - February 2026
- Added URPay integration details
- Updated test card numbers
- Enhanced security guidelines
- Added Node.js examples
- Updated error codes

**Version 1.5** - September 2025
- Added Apple Pay integration
- Enhanced 3DS 2.0 support
- Added installment payments
- Updated API specifications

**Version 1.0** - January 2025
- Initial release
- Bank hosted integration
- Merchant hosted integration
- Basic card support
- MADA integration

---

**Document Version**: 2.0  
**Last Updated**: February 8, 2026  
**Maintained by**: Al Rajhi Bank Technical Team  
**© 2026 Al Rajhi Bank. All rights reserved.**
