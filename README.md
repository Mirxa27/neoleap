# Al Rajhi Bank Payment Gateway - Neoleap Integration

This repository contains comprehensive documentation for integrating with the Al Rajhi Bank (ARB) Payment Gateway REST APIs. The payment gateway is provided through Neoleap, a fintech subsidiary of Al Rajhi Bank, offering secure and PCI-DSS compliant payment processing solutions for businesses in Saudi Arabia.

## Overview

The Al Rajhi Payment Gateway provides a secure, PCI-DSS compliant platform for processing online payments including:
- **Credit and Debit Card Payments** (Visa, Mastercard, MADA, American Express)
- **Digital Wallets** (Apple Pay, URPay)
- **SADAD Payments**
- **Card-on-File Transactions**
- **Recurring Payments**
- **Credit Card Installments**
- **Split Payments and Payouts**

## Integration Options

### Bank Hosted Integration
- Standard checkout with redirect to ARB payment page
- Faster checkout for returning customers
- iFrame integration for embedded payment experience

### Merchant Hosted Integration
- 3D Secure transactions
- Non-3D Secure transactions
- Card-on-File (tokenization)
- Apple Pay integration

## Documentation

- **[Payment Gateway Guide](PAYMENT_CHANNELS.md)** - Comprehensive guide covering all payment methods and transaction flows
- **[API Reference](docs/API.md)** - Complete REST API endpoints, request/response formats, and authentication
- **[Integration Guide](docs/INTEGRATION.md)** - Step-by-step integration procedures with code examples

## Quick Start

1. Obtain your Tranportal ID and password from the ARB Merchant Portal
2. Review the [Integration Guide](docs/INTEGRATION.md) for setup instructions
3. Implement the Payment Token Generation API
4. Test using sandbox environment with test cards
5. Follow security guidelines and encryption requirements

## Key Features

✅ **PCI-DSS Level 1 Compliant** - Secure payment processing  
✅ **3D Secure 2.0** - Enhanced authentication for card payments  
✅ **Multiple Payment Methods** - Cards, wallets, SADAD, installments  
✅ **Tokenization** - Secure card-on-file for repeat customers  
✅ **Webhook Notifications** - Real-time transaction status updates  
✅ **Split Payments** - Distribute funds to multiple accounts  
✅ **Saudi Riyal (SAR)** - Primary currency support

## Security

All API communications use:
- **AES Encryption** with CBC mode and PKCS5Padding
- **HTTPS/TLS** for data transmission
- **Request/Response Encryption** using merchant-specific keys
- **Webhook Signature Verification** for secure notifications

## Support

- **Merchant Portal**: Access transaction reports and account settings
- **Technical Documentation**: v1.31 (December 2024)
- **API Environment**: REST-based with JSON payloads

## About Neoleap

Neoleap is Al Rajhi Bank's fintech subsidiary, providing innovative digital payment solutions and gateway services for merchants across Saudi Arabia.

---

**Document Version**: Based on ARB Payment Gateway REST API Integration Doc v1.31  
**Last Updated**: February 2026  
**Copyright**: © 2024 Al Rajhi Bank. All rights reserved.