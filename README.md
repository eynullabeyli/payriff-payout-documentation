# Payriff API Documentation

REST API reference for the Payriff payment gateway — Payout, Deposit, and Invoice services.

**Base URL**: `https://api.payriff.com` (replace with your environment URL)  
**API Versions**: v2, v3

---

## Authentication

All requests require the `Authorization` header with your API token.

```
Authorization: <YOUR_API_TOKEN>
```

| Credential    | Description                                      |
|---------------|--------------------------------------------------|
| API Token     | Bearer token for authenticating all API requests  |
| Merchant ID   | Your merchant identifier, passed in request bodies where required |

---

## Payout APIs (v3)

### 1. Check Cardholder

Validate whether a card PAN belongs to a valid cardholder before initiating a payout.

```http
POST /v3/payout/check-cardholder
```

**Headers:**

```
Authorization: <YOUR_API_TOKEN>
Content-Type: application/json
```

**Request Body:**

```json
{
  "cardPan": "4169741234567890"
}
```

| Field     | Type   | Required | Description   |
|-----------|--------|----------|---------------|
| `cardPan` | String | Yes      | Full card PAN |

**Response:**

```json
{
  "responseId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "payload": "string"
}
```

| Field        | Type   | Description             |
|--------------|--------|-------------------------|
| `responseId` | String | Unique response ID      |
| `payload`    | String | Cardholder payload data |

---

### 2. Process Payout

Execute a P2P payout transfer to a recipient card.

```http
POST /v3/payout
```

**Headers:**

```
Authorization: <YOUR_API_TOKEN>
Content-Type: application/json
```

**Request Body:**

```json
{
  "merchant": "YOUR_MERCHANT_ID",
  "body": {
    "bankName": "Kapital Bank",
    "fullName": "John Doe",
    "description": "Payment transfer",
    "finCode": "ABC1234",
    "transferAmount": 150.00,
    "requestRrn": "RRN123456",
    "cardPan": "4169741234567890"
  }
}
```

| Field                 | Type   | Required | Description                    |
|-----------------------|--------|----------|--------------------------------|
| `merchant`            | String | Yes      | Your merchant ID               |
| `body.bankName`       | String | Yes      | Recipient bank name            |
| `body.fullName`       | String | Yes      | Recipient full name            |
| `body.description`    | String | No       | Transaction description        |
| `body.finCode`        | String | Yes      | Financial identification code  |
| `body.transferAmount` | double | Yes      | Amount to transfer             |
| `body.requestRrn`     | String | Yes      | Unique Request Reference Number (RRN) |
| `body.cardPan`        | String | Yes      | Recipient card PAN             |

**Success Response:**

```json
{
  "message": "Payout processed successfully"
}
```

**Error Response (4xx):**

```json
{
  "message": "Error description from Payriff"
}
```

---

### 3. Get Payout Info

Retrieve payout transaction details by RRN or payout ID.

```http
GET /v3/payout/info/{identifier}
```

**Headers:**

```
Authorization: <YOUR_API_TOKEN>
```

**Path Parameters:**

| Parameter    | Type          | Description                      |
|--------------|---------------|----------------------------------|
| `identifier` | String / Long | RRN (string) or payout ID (long) |

**Response:**

```json
{
  "code": "00000",
  "message": "Success",
  "route": "string",
  "internalMessage": "string",
  "responseId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "payload": {
    "bankName": "Kapital Bank",
    "state": "SUCCESS",
    "cardPan": "416974******7890",
    "transferAmount": 150.00,
    "appId": 12345,
    "createdDate": "2026-04-13T10:30:00",
    "transferType": "PAYOUT",
    "description": "Payment transfer",
    "fullName": "John Doe",
    "formattedDate": "13.04.2026 10:30"
  }
}
```

**Response Fields:**

| Field             | Type   | Description           |
|-------------------|--------|-----------------------|
| `code`            | String | Response code         |
| `message`         | String | Response message      |
| `route`           | String | Route info            |
| `internalMessage` | String | Internal message      |
| `responseId`      | String | Unique response ID    |

**Payload Fields:**

| Field            | Type   | Description           |
|------------------|--------|-----------------------|
| `bankName`       | String | Bank name             |
| `state`          | String | Transaction state     |
| `cardPan`        | String | Masked card PAN       |
| `transferAmount` | double | Transfer amount       |
| `appId`          | int    | Application ID        |
| `createdDate`    | String | ISO creation date     |
| `transferType`   | String | Transfer type         |
| `description`    | String | Description           |
| `fullName`       | String | Recipient name        |
| `formattedDate`  | String | Formatted date string |

**Transaction States:**

| State         | Meaning                |
|---------------|------------------------|
| `SUCCESS`     | Completed successfully |
| `NOT_FOUND`   | Transaction not found  |
| `IN_PROGRESS` | Still processing       |
| `FAIL`        | Transaction failed     |

---

### 4. Search Payouts

Search and paginate through payout transactions, optionally filtered by FIN code.

```http
POST /v3/payout/by-search
```

**Headers:**

```
Authorization: <YOUR_API_TOKEN>
Content-Type: application/json
```

**Request Body:**

```json
{
  "page": 0,
  "size": 10,
  "finCode": "ABC1234"
}
```

| Field     | Type    | Required | Default | Description                   |
|-----------|---------|----------|---------|-------------------------------|
| `page`    | Integer | No       | `0`     | Page number (zero-based)      |
| `size`    | Integer | No       | `10`    | Number of results per page    |
| `finCode` | String  | No       | `null`  | Filter by FIN code            |

**Response:**

```json
{
  "content": [
    {
      "id": 1,
      "requestRrn": "RRN123456",
      "transferAmount": 150.00,
      "amountWithFee": 152.50,
      "fee": 2.50,
      "createdDate": "2026-04-13T10:30:00",
      "state": "COMPLETED",
      "cardPan": "416974******7890",
      "finCode": "ABC1234",
      "fullName": "John Doe",
      "description": "Payment transfer"
    }
  ],
  "number": 0,
  "size": 10,
  "totalElements": 100,
  "totalPages": 10,
  "last": false,
  "first": true,
  "numberOfElements": 10,
  "empty": false
}
```

**Content Item Fields:**

| Field            | Type   | Description          |
|------------------|--------|----------------------|
| `id`             | Long   | Payout ID            |
| `requestRrn`     | String | Request RRN          |
| `transferAmount` | Double | Transfer amount      |
| `amountWithFee`  | Double | Amount including fee |
| `fee`            | Double | Fee charged          |
| `createdDate`    | String | Creation date        |
| `state`          | String | Transaction state    |
| `cardPan`        | String | Masked card PAN      |
| `finCode`        | String | Financial code       |
| `fullName`       | String | Recipient name       |
| `description`    | String | Description          |

**Pagination Fields:**

| Field              | Type    | Description                     |
|--------------------|---------|---------------------------------|
| `number`           | Integer | Current page number             |
| `size`             | Integer | Page size                       |
| `totalElements`    | Long    | Total number of records         |
| `totalPages`       | Integer | Total number of pages           |
| `last`             | Boolean | Whether this is the last page   |
| `first`            | Boolean | Whether this is the first page  |
| `numberOfElements` | Integer | Number of elements on this page |
| `empty`            | Boolean | Whether the page is empty       |

---

### 5. Check Card Info by BIN

Look up card information (bank, payment system, card product) using the BIN (first 6 digits of the card number).

```http
GET /v3/card/check-card-info/{cardBin}
```

**Headers:**

```
Authorization: <YOUR_API_TOKEN>
```

**Path Parameters:**

| Parameter | Type   | Description                  |
|-----------|--------|------------------------------|
| `cardBin` | String | First 6 digits of card (BIN) |

**Response:**

```json
[
  {
    "bankName": "KBAZ",
    "paymentSystem": "VISA",
    "cardProduct": "CLASSIC",
    "cardBin": "416974",
    "bankNameStr": "Kapital Bank"
  }
]
```

> Note: Response is an array. Use the first element.

| Field           | Type   | Description                    |
|-----------------|--------|--------------------------------|
| `bankName`      | String | Bank code                      |
| `paymentSystem` | String | Payment system (VISA, MC etc.) |
| `cardProduct`   | String | Card product type              |
| `cardBin`       | String | Card BIN                       |
| `bankNameStr`   | String | Bank display name              |

---

### 6. Get Payout Receipt (PDF)

Download a PDF receipt for a completed payout transaction.

```http
GET /v3/payout/receipt/{rrn}
```

**Headers:**

```
Authorization: <YOUR_API_TOKEN>
Accept: application/pdf
```

**Path Parameters:**

| Parameter | Type   | Description              |
|-----------|--------|--------------------------|
| `rrn`     | String | Request Reference Number |

**Response:**

| Header              | Value                                        |
|---------------------|----------------------------------------------|
| `Content-Type`      | `application/pdf`                            |
| `Content-Disposition` | `attachment; filename=payout_receipt_{rrn}.pdf` |

Body contains binary PDF data.

---

## Deposit API (v2)

### 7. Check Deposit Balance

Retrieve the current payout account balance and merchant information.

```http
GET /v2/deposit
```

**Headers:**

```
Authorization: <YOUR_API_TOKEN>
```

**Response:**

```json
{
  "code": "00000",
  "message": "Operation performed successfully",
  "route": "dashboard/main",
  "internalMessage": null,
  "responseId": "f81d4fae7dec11d0a76500a0c91e6bf6",
  "responseDetails": null,
  "payload": {
    "name": "Acme Payments",
    "merchantId": "MR1054721",
    "depositBalance": 18750.45
  }
}
```

**Response Fields:**

| Field                    | Type   | Description                       |
|--------------------------|--------|-----------------------------------|
| `code`                   | String | Response code (`00000` = success) |
| `message`                | String | Operation result message          |
| `route`                  | String | Route info                        |
| `internalMessage`        | String | Internal message (nullable)       |
| `responseId`             | String | Unique response ID                |
| `responseDetails`        | Object | Additional details (nullable)     |
| `payload.name`           | String | Merchant display name             |
| `payload.merchantId`     | String | Merchant ID                       |
| `payload.depositBalance` | double | Current available balance         |

---

## Invoice APIs (v2)

### 8. Create Invoice

Create a payment invoice for collecting payments from customers.

```http
POST /v2/invoices
```

**Headers:**

```
Authorization: <YOUR_API_TOKEN>
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "merchant": "YOUR_MERCHANT_ID",
  "body": {
    "amount": 100.00,
    "currencyType": "AZN",
    "description": "Payment for order #123",
    "languageType": "AZ",
    "approveURL": "https://example.com/success",
    "cancelURL": "https://example.com/cancel",
    "declineURL": "https://example.com/decline",
    "customMessage": "Thank you for your payment",
    "email": "customer@example.com",
    "expireDate": "2026-04-20T23:59:59",
    "fullName": "John Doe",
    "installmentPeriod": 3,
    "installmentProductType": "PRODUCT_TYPE",
    "phoneNumber": "+994501234567",
    "sendSms": true,
    "sendWhatsapp": false,
    "sendEmail": true,
    "amountDynamic": false,
    "directPay": false,
    "metadata": {
      "orderId": "ORD-123",
      "customField": "value"
    }
  }
}
```

| Field                         | Type               | Required | Description                         |
|-------------------------------|--------------------| ---------|-------------------------------------|
| `merchant`                    | String             | Yes      | Your merchant ID                    |
| `body.amount`                 | Double             | Yes      | Invoice amount                      |
| `body.currencyType`           | String             | Yes      | Currency code (AZN, USD, EUR)       |
| `body.description`            | String             | No       | Invoice description                 |
| `body.languageType`           | String             | No       | Language (AZ, EN, RU)               |
| `body.approveURL`             | String             | No       | Redirect URL on payment approval    |
| `body.cancelURL`              | String             | No       | Redirect URL on payment cancellation|
| `body.declineURL`             | String             | No       | Redirect URL on payment decline     |
| `body.customMessage`          | String             | No       | Custom message to display           |
| `body.email`                  | String             | No       | Customer email address              |
| `body.expireDate`             | String (DateTime)  | No       | Invoice expiration (ISO 8601)       |
| `body.fullName`               | String             | No       | Customer full name                  |
| `body.installmentPeriod`      | Integer            | No       | Installment period in months        |
| `body.installmentProductType` | String             | No       | Installment product type            |
| `body.phoneNumber`            | String             | No       | Customer phone number               |
| `body.sendSms`                | Boolean            | No       | Send SMS notification               |
| `body.sendWhatsapp`           | Boolean            | No       | Send WhatsApp notification          |
| `body.sendEmail`              | Boolean            | No       | Send email notification             |
| `body.amountDynamic`          | Boolean            | No       | Allow customer to change amount     |
| `body.directPay`              | Boolean            | No       | Direct payment without redirect     |
| `body.metadata`               | Map<String,String> | No       | Custom key-value metadata           |

**Response:** JSON (structure varies by Payriff).

---

### 9. Get Invoice Info

Retrieve invoice details by UUID.

```http
POST /v2/get-invoice
```

**Headers:**

```
Authorization: <YOUR_API_TOKEN>
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "merchant": "YOUR_MERCHANT_ID",
  "body": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
  }
}
```

| Field       | Type   | Required | Description      |
|-------------|--------|----------|------------------|
| `merchant`  | String | Yes      | Your merchant ID |
| `body.uuid` | String | Yes      | Invoice UUID     |

**Response:** JSON (structure varies by Payriff).

---

## Endpoint Summary

| #  | Method | Path                                 | Version | Description          |
|----|--------|--------------------------------------|---------|----------------------|
| 1  | POST   | `/v3/payout/check-cardholder`        | v3      | Validate cardholder  |
| 2  | POST   | `/v3/payout`                         | v3      | Process payout       |
| 3  | GET    | `/v3/payout/info/{identifier}`       | v3      | Get payout info      |
| 4  | POST   | `/v3/payout/by-search`               | v3      | Search payouts       |
| 5  | GET    | `/v3/card/check-card-info/{cardBin}` | v3      | Card info by BIN     |
| 6  | GET    | `/v3/payout/receipt/{rrn}`           | v3      | Download receipt PDF |
| 7  | GET    | `/v2/deposit`                        | v2      | Check balance        |
| 8  | POST   | `/v2/invoices`                       | v2      | Create invoice       |
| 9  | POST   | `/v2/get-invoice`                    | v2      | Get invoice info     |

---

## Error Handling

| HTTP Status | Description                                          |
|-------------|------------------------------------------------------|
| `200`       | Success                                              |
| `400`       | Bad request — invalid parameters or business error   |
| `401`       | Unauthorized — missing or invalid API token          |
| `404`       | Resource not found                                   |
| `4xx`       | Client error — `message` field contains error detail |
| `5xx`       | Server error                                         |

Error responses include a `message` field describing the issue:

```json
{
  "message": "Error description"
}
```
