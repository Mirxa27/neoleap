# Al Rajhi Bank (ARB) Payment Gateway API Reference

## Table of Contents
- [Base URLs](#base-urls)
- [Authentication](#authentication)
- [API Endpoints](#api-endpoints)
- [Request/Response Formats](#requestresponse-formats)
- [Parameters Reference](#parameters-reference)
- [Webhooks](#webhooks)
- [Error Handling](#error-handling)
- [Transaction Status Values](#transaction-status-values)
- [Rate Limits](#rate-limits)

---

## Base URLs

### Sandbox Environment
- **Payment API**: `https://digitalpayments.alrajhibank.com.sa:2443/pg/payment/tranportal/api`
- **Inquiry API**: `https://digitalpayments.alrajhibank.com.sa:2443/pg/payment/tranportal/inquiry`
- **Refund API**: `https://digitalpayments.alrajhibank.com.sa:2443/pg/payment/tranportal/refund`
- **Void API**: `https://digitalpayments.alrajhibank.com.sa:2443/pg/payment/tranportal/void`
- **Capture API**: `https://digitalpayments.alrajhibank.com.sa:2443/pg/payment/tranportal/capture`
- **Card Registration**: `https://digitalpayments.alrajhibank.com.sa:2443/pg/payment/tranportal/cardregistration`
- **Card Deregistration**: `https://digitalpayments.alrajhibank.com.sa:2443/pg/payment/tranportal/cardderegistration`
- **BIN Check API**: `https://digitalpayments.alrajhibank.com.sa:2443/pg/payment/tranportal/issuercountry`
- **Installment Plans**: `https://digitalpayments.alrajhibank.com.sa:2443/pg/payment/tranportal/getinstallmentplans`

### Production Environment
- **Payment API**: `https://digitalpayments.alrajhibank.com.sa/pg/payment/tranportal/api`
- **Inquiry API**: `https://digitalpayments.alrajhibank.com.sa/pg/payment/tranportal/inquiry`
- **Refund API**: `https://digitalpayments.alrajhibank.com.sa/pg/payment/tranportal/refund`
- **Void API**: `https://digitalpayments.alrajhibank.com.sa/pg/payment/tranportal/void`
- **Capture API**: `https://digitalpayments.alrajhibank.com.sa/pg/payment/tranportal/capture`
- **Card Registration**: `https://digitalpayments.alrajhibank.com.sa/pg/payment/tranportal/cardregistration`
- **Card Deregistration**: `https://digitalpayments.alrajhibank.com.sa/pg/payment/tranportal/cardderegistration`
- **BIN Check API**: `https://digitalpayments.alrajhibank.com.sa/pg/payment/tranportal/issuercountry`
- **Installment Plans**: `https://digitalpayments.alrajhibank.com.sa/pg/payment/tranportal/getinstallmentplans`

---

## Authentication

All API requests require authentication using Tranportal ID and Password with AES encryption.

### Authentication Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tranportalId` | String | Yes | Merchant's Tranportal ID provided by ARB |
| `tranportalPassword` | String | Yes | Merchant's Tranportal Password (AES encrypted) |

### AES Encryption

The `tranportalPassword` must be encrypted using **AES-256-CBC** encryption before sending in requests.

**Encryption Specifications:**
- **Algorithm**: AES-256-CBC
- **Key Size**: 256 bits (32 bytes)
- **IV (Initialization Vector)**: 16 bytes (provided by ARB)
- **Padding**: PKCS7
- **Output Format**: Base64 encoded string

**Example (Node.js):**
```javascript
const crypto = require('crypto');

function encryptPassword(password, key, iv) {
  const cipher = crypto.createCipheriv('aes-256-cbc', Buffer.from(key), Buffer.from(iv));
  let encrypted = cipher.update(password, 'utf8', 'base64');
  encrypted += cipher.final('base64');
  return encrypted;
}
```

**Example (Python):**
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import base64

def encrypt_password(password, key, iv):
    cipher = AES.new(key.encode(), AES.MODE_CBC, iv.encode())
    encrypted = cipher.encrypt(pad(password.encode(), AES.block_size))
    return base64.b64encode(encrypted).decode()
```

### Request Headers

All API requests must include:

```
Content-Type: application/json
Accept: application/json
```

---

## API Endpoints

### 1. Payment Token Generation API

Generate a payment token to initiate a transaction.

**Endpoint:** `POST /pg/payment/tranportal/api`

**Request Body:**
```json
{
  "tranportalId": "12345678",
  "tranportalPassword": "AES_ENCRYPTED_PASSWORD",
  "action": "1",
  "merchantIdentifier": "MerchantID123",
  "amount": "100.00",
  "currency": "SAR",
  "customerEmail": "customer@example.com",
  "requestPhrase": "MerchantSecretKey",
  "language": "en",
  "returnUrl": "https://merchant.com/return",
  "merchantReference": "ORD-2024-001",
  "orderDescription": "Order for Product XYZ",
  "customerName": "Ahmed Ali",
  "customerPhone": "966501234567",
  "customerAddress": "Riyadh, Saudi Arabia",
  "tokenName": "CUST_TOKEN_001",
  "paymentOption": "VISA",
  "sadad": {
    "sadad": "1"
  },
  "installments": "STANDALONE",
  "issuerCode": "ARB",
  "eci": "ECOMMERCE"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "paymentToken": "ABC123XYZ456TOKEN789",
  "paymentUrl": "https://digitalpayments.alrajhibank.com.sa/payment?token=ABC123XYZ456TOKEN789",
  "expiryTime": "2024-02-08T15:30:00Z",
  "merchantReference": "ORD-2024-001"
}
```

---

### 2. Transaction Inquiry API

Query the status of a transaction.

**Endpoint:** `POST /pg/payment/tranportal/inquiry`

**Request Body:**
```json
{
  "tranportalId": "12345678",
  "tranportalPassword": "AES_ENCRYPTED_PASSWORD",
  "action": "10",
  "merchantIdentifier": "MerchantID123",
  "requestPhrase": "MerchantSecretKey",
  "merchantReference": "ORD-2024-001",
  "language": "en"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "transactionStatus": "CAPTURED",
  "authorizationCode": "123456",
  "merchantReference": "ORD-2024-001",
  "transactionId": "TXN123456789",
  "amount": "100.00",
  "currency": "SAR",
  "paymentOption": "VISA",
  "cardNumber": "411111******1111",
  "customerEmail": "customer@example.com",
  "cardholderName": "AHMED ALI",
  "eci": "05",
  "fortId": "168234567890",
  "responseCode": "00000",
  "responseMessage": "Success",
  "authorizationMessage": "Approved",
  "transactionTime": "2024-02-08T14:23:45Z",
  "signature": "7cad05f0212ed933c9a5d5dffa31661e"
}
```

---

### 3. Refund API

Refund a captured or partially captured transaction.

**Endpoint:** `POST /pg/payment/tranportal/refund`

**Request Body:**
```json
{
  "tranportalId": "12345678",
  "tranportalPassword": "AES_ENCRYPTED_PASSWORD",
  "action": "4",
  "merchantIdentifier": "MerchantID123",
  "requestPhrase": "MerchantSecretKey",
  "merchantReference": "ORD-2024-001",
  "fortId": "168234567890",
  "amount": "50.00",
  "currency": "SAR",
  "language": "en",
  "orderDescription": "Partial refund for order"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "responseCode": "00000",
  "responseMessage": "Success",
  "merchantReference": "ORD-2024-001",
  "transactionId": "REFUND123456",
  "fortId": "168234567890",
  "amount": "50.00",
  "currency": "SAR",
  "refundedAmount": "50.00",
  "transactionStatus": "REFUNDED",
  "authorizationCode": "789012",
  "signature": "8dbe06f1323fe944d0b6e6eggb42772f"
}
```

---

### 4. Void API

Void an authorized but not yet captured transaction.

**Endpoint:** `POST /pg/payment/tranportal/void`

**Request Body:**
```json
{
  "tranportalId": "12345678",
  "tranportalPassword": "AES_ENCRYPTED_PASSWORD",
  "action": "5",
  "merchantIdentifier": "MerchantID123",
  "requestPhrase": "MerchantSecretKey",
  "merchantReference": "ORD-2024-001",
  "fortId": "168234567890",
  "language": "en",
  "orderDescription": "Void transaction"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "responseCode": "00000",
  "responseMessage": "Success",
  "merchantReference": "ORD-2024-001",
  "transactionId": "VOID123456",
  "fortId": "168234567890",
  "transactionStatus": "VOID",
  "authorizationCode": "345678",
  "signature": "9ecf17g2434gf055e1c7f7fhh53883g"
}
```

---

### 5. Capture API

Capture a previously authorized transaction.

**Endpoint:** `POST /pg/payment/tranportal/capture`

**Request Body:**
```json
{
  "tranportalId": "12345678",
  "tranportalPassword": "AES_ENCRYPTED_PASSWORD",
  "action": "3",
  "merchantIdentifier": "MerchantID123",
  "requestPhrase": "MerchantSecretKey",
  "merchantReference": "ORD-2024-001",
  "fortId": "168234567890",
  "amount": "100.00",
  "currency": "SAR",
  "language": "en",
  "orderDescription": "Capture full amount"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "responseCode": "00000",
  "responseMessage": "Success",
  "merchantReference": "ORD-2024-001",
  "transactionId": "CAPT123456",
  "fortId": "168234567890",
  "amount": "100.00",
  "currency": "SAR",
  "transactionStatus": "CAPTURED",
  "authorizationCode": "901234",
  "signature": "0fah28h3545hg166f2d8g8gii64994h"
}
```

---

### 6. Card Registration API

Register a card for tokenization (recurring payments).

**Endpoint:** `POST /pg/payment/tranportal/cardregistration`

**Request Body:**
```json
{
  "tranportalId": "12345678",
  "tranportalPassword": "AES_ENCRYPTED_PASSWORD",
  "action": "11",
  "merchantIdentifier": "MerchantID123",
  "requestPhrase": "MerchantSecretKey",
  "customerEmail": "customer@example.com",
  "tokenName": "CUST_TOKEN_001",
  "returnUrl": "https://merchant.com/return",
  "language": "en",
  "cardNumber": "4111111111111111",
  "expiryDate": "2512",
  "cardSecurityCode": "123",
  "cardholderName": "AHMED ALI",
  "rememberMe": "YES"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "responseCode": "00000",
  "responseMessage": "Success",
  "tokenName": "CUST_TOKEN_001",
  "cardToken": "TOKEN_ABC123XYZ456",
  "cardNumber": "411111******1111",
  "expiryDate": "2512",
  "cardholderName": "AHMED ALI",
  "paymentOption": "VISA",
  "signature": "1gbj39i4656ih277g3e9h9hjj75005i"
}
```

---

### 7. Card Deregistration API

Remove a registered card token.

**Endpoint:** `POST /pg/payment/tranportal/cardderegistration`

**Request Body:**
```json
{
  "tranportalId": "12345678",
  "tranportalPassword": "AES_ENCRYPTED_PASSWORD",
  "action": "12",
  "merchantIdentifier": "MerchantID123",
  "requestPhrase": "MerchantSecretKey",
  "tokenName": "CUST_TOKEN_001",
  "cardToken": "TOKEN_ABC123XYZ456",
  "language": "en"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "responseCode": "00000",
  "responseMessage": "Card token removed successfully",
  "tokenName": "CUST_TOKEN_001",
  "signature": "2hck40j5767ji388h4f0i0ikk86116j"
}
```

---

### 8. Issuer Country API (Card BIN Check)

Retrieve issuer country information based on card BIN.

**Endpoint:** `POST /pg/payment/tranportal/issuercountry`

**Request Body:**
```json
{
  "tranportalId": "12345678",
  "tranportalPassword": "AES_ENCRYPTED_PASSWORD",
  "action": "13",
  "merchantIdentifier": "MerchantID123",
  "requestPhrase": "MerchantSecretKey",
  "cardBin": "411111",
  "language": "en"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "responseCode": "00000",
  "responseMessage": "Success",
  "cardBin": "411111",
  "issuerCountry": "SA",
  "issuerCountryName": "Saudi Arabia",
  "paymentOption": "VISA",
  "cardType": "CREDIT",
  "signature": "3idl51k6878kj499i5g1j1jll97227k"
}
```

---

### 9. Get Installment Plans API

Retrieve available installment plans for a transaction.

**Endpoint:** `POST /pg/payment/tranportal/getinstallmentplans`

**Request Body:**
```json
{
  "tranportalId": "12345678",
  "tranportalPassword": "AES_ENCRYPTED_PASSWORD",
  "action": "14",
  "merchantIdentifier": "MerchantID123",
  "requestPhrase": "MerchantSecretKey",
  "amount": "1000.00",
  "currency": "SAR",
  "issuerCode": "ARB",
  "language": "en"
}
```

**Response:** `200 OK`
```json
{
  "status": "success",
  "responseCode": "00000",
  "responseMessage": "Success",
  "amount": "1000.00",
  "currency": "SAR",
  "issuerCode": "ARB",
  "installmentPlans": [
    {
      "planCode": "3MONTHS",
      "numberOfInstallments": 3,
      "installmentAmount": "333.33",
      "totalAmount": "1000.00",
      "interestRate": "0.00",
      "fees": "0.00"
    },
    {
      "planCode": "6MONTHS",
      "numberOfInstallments": 6,
      "installmentAmount": "166.67",
      "totalAmount": "1000.00",
      "interestRate": "0.00",
      "fees": "0.00"
    },
    {
      "planCode": "12MONTHS",
      "numberOfInstallments": 12,
      "installmentAmount": "83.33",
      "totalAmount": "1000.00",
      "interestRate": "0.00",
      "fees": "0.00"
    }
  ],
  "signature": "4jem62l7989lk500j6h2k2kmm08338l"
}
```

---

## Parameters Reference

### Payment Token Generation Parameters

| Parameter | Type | M/O/C | Max Length | Description |
|-----------|------|-------|------------|-------------|
| `tranportalId` | String | M | 16 | Merchant's Tranportal ID |
| `tranportalPassword` | String | M | 256 | AES encrypted password |
| `action` | String | M | 2 | Action code: "1" for payment |
| `merchantIdentifier` | String | M | 32 | Merchant identifier |
| `amount` | Decimal | M | 10 | Transaction amount (e.g., "100.00") |
| `currency` | String | M | 3 | Currency code (ISO 4217): SAR, USD, EUR |
| `customerEmail` | String | M | 254 | Customer email address |
| `requestPhrase` | String | M | 128 | Merchant secret key for signature |
| `language` | String | M | 2 | Language code: en, ar |
| `returnUrl` | String | M | 512 | URL to redirect after payment |
| `merchantReference` | String | M | 40 | Unique order reference |
| `orderDescription` | String | O | 150 | Description of the order |
| `customerName` | String | O | 40 | Customer full name |
| `customerPhone` | String | O | 19 | Customer phone with country code |
| `customerAddress` | String | O | 255 | Customer address |
| `tokenName` | String | C | 100 | Token identifier for card registration |
| `paymentOption` | String | O | 10 | Payment method: VISA, MASTERCARD, MADA |
| `sadad` | Object | C | - | SADAD payment details |
| `installments` | String | C | 20 | Installment type: STANDALONE, HOSTED |
| `issuerCode` | String | C | 10 | Issuer bank code |
| `eci` | String | O | 16 | E-commerce indicator: ECOMMERCE, MOTO, RECURRING |
| `command` | String | O | 20 | Transaction command: AUTHORIZATION, PURCHASE |
| `rememberMe` | String | O | 3 | Save card: YES, NO |
| `cardSecurityCode` | String | C | 4 | CVV/CVC code |
| `cardNumber` | String | C | 19 | Card number (PCI compliant) |
| `expiryDate` | String | C | 4 | Card expiry (YYMM format) |
| `cardholderName` | String | C | 50 | Name on card |
| `settlementReference` | String | O | 34 | Settlement reference number |
| `merchantExtra` | String | O | 999 | Additional merchant data (JSON) |
| `merchantExtra1` | String | O | 250 | Custom field 1 |
| `merchantExtra2` | String | O | 250 | Custom field 2 |
| `merchantExtra3` | String | O | 250 | Custom field 3 |
| `merchantExtra4` | String | O | 250 | Custom field 4 |

**Legend:**
- **M** = Mandatory
- **O** = Optional
- **C** = Conditional (required in specific scenarios)

### Transaction Inquiry Parameters

| Parameter | Type | M/O/C | Max Length | Description |
|-----------|------|-------|------------|-------------|
| `tranportalId` | String | M | 16 | Merchant's Tranportal ID |
| `tranportalPassword` | String | M | 256 | AES encrypted password |
| `action` | String | M | 2 | Action code: "10" for inquiry |
| `merchantIdentifier` | String | M | 32 | Merchant identifier |
| `requestPhrase` | String | M | 128 | Merchant secret key |
| `merchantReference` | String | M | 40 | Order reference to query |
| `language` | String | M | 2 | Language code: en, ar |
| `fortId` | String | C | 20 | ARB transaction ID (alternative to merchantReference) |

### Refund Parameters

| Parameter | Type | M/O/C | Max Length | Description |
|-----------|------|-------|------------|-------------|
| `tranportalId` | String | M | 16 | Merchant's Tranportal ID |
| `tranportalPassword` | String | M | 256 | AES encrypted password |
| `action` | String | M | 2 | Action code: "4" for refund |
| `merchantIdentifier` | String | M | 32 | Merchant identifier |
| `requestPhrase` | String | M | 128 | Merchant secret key |
| `merchantReference` | String | M | 40 | Original order reference |
| `fortId` | String | M | 20 | ARB transaction ID |
| `amount` | Decimal | M | 10 | Refund amount (partial or full) |
| `currency` | String | M | 3 | Currency code |
| `language` | String | M | 2 | Language code: en, ar |
| `orderDescription` | String | O | 150 | Refund description |

### Void Parameters

| Parameter | Type | M/O/C | Max Length | Description |
|-----------|------|-------|------------|-------------|
| `tranportalId` | String | M | 16 | Merchant's Tranportal ID |
| `tranportalPassword` | String | M | 256 | AES encrypted password |
| `action` | String | M | 2 | Action code: "5" for void |
| `merchantIdentifier` | String | M | 32 | Merchant identifier |
| `requestPhrase` | String | M | 128 | Merchant secret key |
| `merchantReference` | String | M | 40 | Original order reference |
| `fortId` | String | M | 20 | ARB transaction ID |
| `language` | String | M | 2 | Language code: en, ar |
| `orderDescription` | String | O | 150 | Void description |

### Capture Parameters

| Parameter | Type | M/O/C | Max Length | Description |
|-----------|------|-------|------------|-------------|
| `tranportalId` | String | M | 16 | Merchant's Tranportal ID |
| `tranportalPassword` | String | M | 256 | AES encrypted password |
| `action` | String | M | 2 | Action code: "3" for capture |
| `merchantIdentifier` | String | M | 32 | Merchant identifier |
| `requestPhrase` | String | M | 128 | Merchant secret key |
| `merchantReference` | String | M | 40 | Original order reference |
| `fortId` | String | M | 20 | ARB transaction ID |
| `amount` | Decimal | M | 10 | Capture amount (partial or full) |
| `currency` | String | M | 3 | Currency code |
| `language` | String | M | 2 | Language code: en, ar |
| `orderDescription` | String | O | 150 | Capture description |

---

## Webhooks

ARB Payment Gateway sends webhook notifications for transaction events to your configured endpoint.

### Configuration

Configure your webhook URL in the ARB merchant portal. The URL must:
- Be HTTPS only
- Accept POST requests
- Return HTTP 200 status within 30 seconds
- Handle duplicate notifications (idempotent processing)

### Webhook Events

| Event Type | Description |
|------------|-------------|
| `payment.authorized` | Payment authorized but not captured |
| `payment.captured` | Payment captured successfully |
| `payment.declined` | Payment declined by issuer |
| `payment.failed` | Payment failed due to error |
| `payment.refunded` | Full or partial refund processed |
| `payment.voided` | Authorization voided |
| `3ds.authenticated` | 3D Secure authentication completed |
| `3ds.failed` | 3D Secure authentication failed |

### Webhook Payload Structure

**Headers:**
```
Content-Type: application/json
X-ARB-Signature: sha256_hmac_signature
X-ARB-Event: payment.captured
X-ARB-Timestamp: 1707404625
```

**Body:**
```json
{
  "event": "payment.captured",
  "eventId": "evt_123456789",
  "timestamp": "2024-02-08T14:23:45Z",
  "data": {
    "merchantReference": "ORD-2024-001",
    "fortId": "168234567890",
    "transactionId": "TXN123456789",
    "amount": "100.00",
    "currency": "SAR",
    "transactionStatus": "CAPTURED",
    "authorizationCode": "123456",
    "responseCode": "00000",
    "responseMessage": "Success",
    "paymentOption": "VISA",
    "cardNumber": "411111******1111",
    "expiryDate": "2512",
    "customerEmail": "customer@example.com",
    "customerName": "Ahmed Ali",
    "customerPhone": "966501234567",
    "eci": "05",
    "orderDescription": "Order for Product XYZ",
    "transactionTime": "2024-02-08T14:23:45Z",
    "signature": "7cad05f0212ed933c9a5d5dffa31661e",
    "merchantExtra": "{\"customField\":\"value\"}",
    "settlementReference": "SETTLE123456"
  }
}
```

### Signature Verification

Verify the webhook signature to ensure authenticity:

**Algorithm:** HMAC-SHA256

**Signature Payload:**
```
{timestamp}.{eventId}.{jsonBody}
```

**Example (Node.js):**
```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  const expectedSignature = hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

// Usage
const timestamp = req.headers['x-arb-timestamp'];
const eventId = req.body.eventId;
const body = JSON.stringify(req.body);
const payload = `${timestamp}.${eventId}.${body}`;
const signature = req.headers['x-arb-signature'];

if (verifyWebhookSignature(payload, signature, merchantSecret)) {
  // Process webhook
}
```

### Retry Logic

If your endpoint fails to respond with HTTP 200:
- **Retry 1**: After 5 minutes
- **Retry 2**: After 15 minutes
- **Retry 3**: After 1 hour
- **Retry 4**: After 6 hours
- **Retry 5**: After 24 hours

After 5 failed attempts, the webhook is marked as failed and no further retries occur.

### Acknowledgement Requirements

Your webhook endpoint must:
1. Return HTTP 200 status within 30 seconds
2. Process notifications asynchronously (don't block response)
3. Validate signature before processing
4. Handle duplicate events (use `eventId` for deduplication)
5. Log all received webhooks for debugging

---

## Error Handling

### Error Response Format

```json
{
  "status": "error",
  "responseCode": "E001",
  "responseMessage": "Invalid merchant credentials",
  "errors": [
    {
      "field": "tranportalId",
      "code": "INVALID_VALUE",
      "message": "Tranportal ID is invalid or inactive"
    }
  ],
  "timestamp": "2024-02-08T14:23:45Z",
  "requestId": "req_abc123xyz456"
}
```

### Common Error Codes

| Error Code | Description | Resolution |
|------------|-------------|------------|
| `E001` | Invalid merchant credentials | Verify tranportalId and password |
| `E002` | Invalid or expired payment token | Generate new payment token |
| `E003` | Insufficient parameters | Check all mandatory fields are present |
| `E004` | Invalid amount format | Amount must be decimal with 2 decimals |
| `E005` | Unsupported currency | Use SAR, USD, EUR, or other supported currencies |
| `E006` | Invalid card number | Check card number format and validity |
| `E007` | Card expired | Card expiry date is in the past |
| `E008` | Invalid CVV | CVV must be 3-4 digits |
| `E009` | Transaction not found | Verify merchantReference or fortId |
| `E010` | Transaction already processed | Cannot reprocess completed transaction |
| `E011` | Refund amount exceeds captured amount | Reduce refund amount |
| `E012` | Cannot void captured transaction | Use refund API instead |
| `E013` | Signature mismatch | Check request signature calculation |
| `E014` | Rate limit exceeded | Wait before retrying |
| `E015` | Service temporarily unavailable | Retry after delay |

### Authorization Response Codes

| Code | Status | Description |
|------|--------|-------------|
| `00000` | Success | Transaction approved |
| `00001` | Declined | Refer to card issuer |
| `00002` | Declined | Refer to card issuer (special condition) |
| `00003` | Declined | Invalid merchant |
| `00004` | Declined | Pick up card |
| `00005` | Declined | Do not honor |
| `00006` | Error | Error in processing |
| `00007` | Declined | Pick up card (special condition) |
| `00012` | Declined | Invalid transaction |
| `00013` | Declined | Invalid amount |
| `00014` | Declined | Invalid card number |
| `00015` | Declined | No such issuer |
| `00030` | Declined | Format error |
| `00041` | Declined | Lost card |
| `00043` | Declined | Stolen card |
| `00051` | Declined | Insufficient funds |
| `00054` | Declined | Expired card |
| `00055` | Declined | Incorrect PIN |
| `00057` | Declined | Transaction not permitted to cardholder |
| `00058` | Declined | Transaction not permitted to terminal |
| `00061` | Declined | Exceeds withdrawal amount limit |
| `00062` | Declined | Restricted card |
| `00063` | Declined | Security violation |
| `00065` | Declined | Exceeds withdrawal frequency limit |
| `00091` | Timeout | Issuer unavailable |
| `00096` | Error | System malfunction |

### 3D Secure Response Codes

| ECI Value | Description | Liability Shift |
|-----------|-------------|-----------------|
| `05` | 3DS authenticated (Visa) | Yes |
| `06` | 3DS attempted (Visa) | No |
| `07` | Non-3DS transaction (Visa) | No |
| `02` | 3DS authenticated (Mastercard) | Yes |
| `01` | 3DS attempted (Mastercard) | No |
| `00` | Non-3DS transaction (Mastercard) | No |

### HTTP Status Codes

| Status Code | Description |
|-------------|-------------|
| `200` | Success - Request processed |
| `400` | Bad Request - Invalid parameters |
| `401` | Unauthorized - Authentication failed |
| `403` | Forbidden - Access denied |
| `404` | Not Found - Endpoint not found |
| `422` | Unprocessable Entity - Validation failed |
| `429` | Too Many Requests - Rate limit exceeded |
| `500` | Internal Server Error - Gateway error |
| `502` | Bad Gateway - Upstream service error |
| `503` | Service Unavailable - Maintenance mode |
| `504` | Gateway Timeout - Request timeout |

---

## Transaction Status Values

| Status | Description | Next Actions |
|--------|-------------|--------------|
| `PENDING` | Transaction initiated but not completed | Wait for customer action |
| `PROCESSING` | Transaction being processed | Poll inquiry API |
| `AUTHORIZED` | Payment authorized, awaiting capture | Capture or void |
| `CAPTURED` | Payment captured successfully | Can refund |
| `APPROVED` | Payment approved and settled | Can refund |
| `DECLINED` | Payment declined by issuer | No further action |
| `FAILED` | Payment failed due to error | Retry or investigate |
| `VOIDED` | Authorization voided | No further action |
| `REFUNDED` | Payment refunded (full) | No further action |
| `PARTIAL_REFUNDED` | Payment partially refunded | Can refund remaining |
| `EXPIRED` | Payment token expired | Generate new token |
| `CANCELLED` | Payment cancelled by customer | No further action |
| `3DS_AUTHENTICATED` | 3D Secure authentication successful | Continue payment |
| `3DS_FAILED` | 3D Secure authentication failed | Cannot proceed |
| `ON_HOLD` | Transaction under review | Wait for manual review |
| `CHARGEBACK` | Chargeback initiated | Dispute process |

---

## Rate Limits

### Request Limits

| API Endpoint | Rate Limit | Burst Limit |
|--------------|------------|-------------|
| Payment Token Generation | 100 req/min | 500 req/hour |
| Transaction Inquiry | 200 req/min | 1000 req/hour |
| Refund API | 50 req/min | 200 req/hour |
| Void API | 50 req/min | 200 req/hour |
| Capture API | 100 req/min | 500 req/hour |
| Card Registration | 20 req/min | 100 req/hour |
| Card Deregistration | 20 req/min | 100 req/hour |
| BIN Check API | 300 req/min | 1500 req/hour |
| Installment Plans | 100 req/min | 500 req/hour |

### Rate Limit Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 85
X-RateLimit-Reset: 1707404700
X-RateLimit-Retry-After: 45
```

### Rate Limit Exceeded Response

```json
{
  "status": "error",
  "responseCode": "E014",
  "responseMessage": "Rate limit exceeded",
  "retryAfter": 45,
  "limit": 100,
  "window": "1 minute"
}
```

### Best Practices

1. **Implement exponential backoff** when retrying failed requests
2. **Cache inquiry responses** to reduce API calls
3. **Use webhooks** instead of polling for transaction status
4. **Batch operations** when possible
5. **Monitor rate limit headers** and adjust request frequency
6. **Implement circuit breakers** for automatic retry management

---

## Best Practices

### Security
- Always use HTTPS for all API calls
- Store credentials securely (use environment variables or secrets manager)
- Encrypt sensitive data before transmission
- Validate webhook signatures
- Implement request signing for added security
- Use PCI-compliant card handling (never store CVV)
- Rotate encryption keys periodically

### Performance
- Implement connection pooling for API calls
- Use asynchronous processing for webhooks
- Cache static data (e.g., installment plans)
- Implement retry logic with exponential backoff
- Monitor API response times and set appropriate timeouts

### Error Handling
- Log all API requests and responses
- Implement proper error handling for all scenarios
- Use transaction inquiry API to verify status
- Set up monitoring and alerting for failed transactions
- Implement graceful degradation for service outages

### Testing
- Test in sandbox environment before production
- Test all payment scenarios (success, decline, timeout)
- Test webhook handling and retry logic
- Test signature verification
- Test rate limiting behavior
- Perform load testing before launch

---

## Support

### Technical Support
- **Email**: digitalsupport@alrajhibank.com.sa
- **Phone**: +966 11 211 6677
- **Hours**: 24/7 for critical issues

### Integration Support
- **Email**: merchantsupport@alrajhibank.com.sa
- **Documentation**: https://digitalpayments.alrajhibank.com.sa/docs
- **Developer Portal**: https://developer.alrajhibank.com.sa

### Incident Reporting
For security incidents or vulnerabilities:
- **Email**: security@alrajhibank.com.sa
- **Response Time**: Within 4 hours

---

## Changelog

### Version 2.1.0 (2024-02-08)
- Added Installment Plans API
- Enhanced 3D Secure 2.0 support
- Added MADA card support
- Improved webhook retry logic
- Added BIN check API

### Version 2.0.0 (2023-09-15)
- Complete API redesign
- Added card tokenization
- Enhanced security with AES-256 encryption
- Added webhook support
- Improved error handling

### Version 1.0.0 (2022-01-01)
- Initial API release
