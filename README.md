# Payriff Host API Documentation

Documentation of external Payriff API endpoints consumed by the `payriff-payout` service.

**Base URL**: Configured via `payriff.url` property (loaded from Vault)  
**API Versions**: v2, v3

---

## Authentication

All requests to the Payriff host require the `Authorization` header with the service token.

```
Authorization: <PAYRIFF_TOKEN>
```

- **Token**: Configured via `payriffpayment.token` (loaded from Vault)
- **Merchant ID**: Configured via `payriffpayment.merchantId` (embedded in request bodies where needed)

---

## Payout APIs (v3)

### 1. Check Cardholder

Validate whether a card PAN belongs to a valid cardholder before initiating a payout.

```
POST {payriff.url}/v3/payout/check-cardholder
```

**Headers:**

| Header          | Value              |
|-----------------|--------------------|
| `Authorization` | `<PAYRIFF_TOKEN>`  |
| `Content-Type`  | `application/json` |

**Request Body:**

```json
{
  "cardPan": "4169741234567890"
}
```

| Field     | Type   | Description    |
|-----------|--------|----------------|
| `cardPan` | String | Full card PAN  |

**Response:**

```json
{
  "responseId": "string",
  "payload": "string"
}
```

| Field        | Type   | Description             |
|--------------|--------|-------------------------|
| `responseId` | String | Payriff response ID     |
| `payload`    | String | Cardholder payload data |

---

### 2. Process Payout

Execute a P2P payout transfer to a recipient card.

```
POST {payriff.url}/v3/payout
```

**Headers:**

| Header          | Value              |
|-----------------|--------------------|
| `Authorization` | `<PAYRIFF_TOKEN>`  |
| `Content-Type`  | `application/json` |

**Request Body:**

```json
{
  "merchant": "<MERCHANT_ID>",
  "body": {
    "bankName": "Kapital Bank",
    "fullName": "John Doe",
    "description": "Loan disbursement",
    "finCode": "ABC1234",
    "transferAmount": 150.00,
    "requestRrn": "RRN123456",
    "cardPan": "4169741234567890"
  }
}
```

| Field                | Type   | Description                    |
|----------------------|--------|--------------------------------|
| `merchant`           | String | Merchant ID                    |
| `body.bankName`      | String | Recipient bank name            |
| `body.fullName`      | String | Recipient full name            |
| `body.description`   | String | Transaction description        |
| `body.finCode`       | String | Financial identification code  |
| `body.transferAmount`| double | Amount to transfer             |
| `body.requestRrn`    | String | Request Reference Number (RRN) |
| `body.cardPan`       | String | Recipient card PAN             |

**Response:**

```json
{
  "message": "string"
}
```

**Error Response (4xx):**

The `message` field from the Payriff error response body is extracted and returned.

---

### 3. Get Payout Info

Retrieve payout transaction details by RRN or payout ID. Same endpoint is used for both.

```
GET {payriff.url}/v3/payout/info/{identifier}
```

**Headers:**

| Header          | Value             |
|-----------------|-------------------|
| `Authorization` | `<PAYRIFF_TOKEN>` |

**Path Parameters:**

| Parameter    | Type          | Description                       |
|--------------|---------------|-----------------------------------|
| `identifier` | String / Long | RRN (string) or payout ID (long)  |

**Response:**

```json
{
  "code": "string",
  "message": "string",
  "route": "string",
  "internalMessage": "string",
  "responseId": "string",
  "payload": {
    "bankName": "string",
    "state": "SUCCESS",
    "cardPan": "416974******7890",
    "transferAmount": 150.00,
    "appId": 12345,
    "createdDate": "2026-04-13T10:30:00",
    "transferType": "string",
    "description": "string",
    "fullName": "John Doe",
    "formattedDate": "13.04.2026 10:30"
  }
}
```

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

| State         | Meaning                    |
|---------------|----------------------------|
| `SUCCESS`     | Completed successfully     |
| `NOT_FOUND`   | Transaction not found      |
| `IN_PROGRESS` | Still processing           |
| `FAIL`        | Transaction failed         |

---

### 4. Search Payouts

Search and paginate through payout transactions, optionally filtered by FIN code.

```
POST {payriff.url}/v3/payout/by-search
```

**Headers:**

| Header          | Value              |
|-----------------|--------------------|
| `Authorization` | `<PAYRIFF_TOKEN>`  |
| `Content-Type`  | `application/json` |

**Request Body:**

```json
{
  "page": 0,
  "size": 10,
  "finCode": "ABC1234"
}
```

| Field     | Type    | Default | Description                     |
|-----------|---------|---------|---------------------------------|
| `page`    | Integer | `0`     | Page number (zero-based)        |
| `size`    | Integer | `10`    | Page size                       |
| `finCode` | String  | `null`  | Filter by FIN code (optional)   |

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
      "description": "Loan disbursement"
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

| Field            | Type   | Description           |
|------------------|--------|-----------------------|
| `id`             | Long   | Payout ID             |
| `requestRrn`     | String | Request RRN           |
| `transferAmount` | Double | Transfer amount       |
| `amountWithFee`  | Double | Amount including fee  |
| `fee`            | Double | Fee charged           |
| `createdDate`    | String | Creation date         |
| `state`          | String | Transaction state     |
| `cardPan`        | String | Masked card PAN       |
| `finCode`        | String | Financial code        |
| `fullName`       | String | Recipient name        |
| `description`    | String | Description           |

---

### 5. Check Card Info by BIN

Look up card information (bank, payment system, product) using the BIN (first 6 digits).

```
GET {payriff.url}/v3/card/check-card-info/{cardBin}
```

**Headers:**

| Header          | Value             |
|-----------------|-------------------|
| `Authorization` | `<PAYRIFF_TOKEN>` |

**Path Parameters:**

| Parameter | Type   | Description                  |
|-----------|--------|------------------------------|
| `cardBin` | String | First 6 digits of card (BIN) |

**Response** (array — first element is used):

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

| Field           | Type   | Description                    |
|-----------------|--------|--------------------------------|
| `bankName`      | String | Bank code                      |
| `paymentSystem` | String | Payment system (VISA, MC etc.) |
| `cardProduct`   | String | Card product type              |
| `cardBin`       | String | Card BIN                       |
| `bankNameStr`   | String | Bank display name              |

---

### 6. Get Payout Receipt PDF

Download the PDF receipt for a completed payout by its RRN.

```
GET {payriff.url}/v3/payout/receipt/{rrn}
```

**Headers:**

| Header          | Value             |
|-----------------|-------------------|
| `Authorization` | `<PAYRIFF_TOKEN>` |
| `Accept`        | `application/pdf` |

**Path Parameters:**

| Parameter | Type   | Description               |
|-----------|--------|---------------------------|
| `rrn`     | String | Request Reference Number  |

**Response:**

- **Content-Type**: `application/pdf`
- **Body**: Binary PDF data

> Note: The service looks up the RRN from the local database using the `requestId`, then fetches the receipt from Payriff using the RRN.

---

## Deposit API (v2)

### 7. Check Deposit Balance

Retrieve the current deposit/payout account balance.

```
GET {payriff.url}/v2/deposit
```

**Headers:**

| Header          | Value             |
|-----------------|-------------------|
| `Authorization` | `<PAYRIFF_TOKEN>` |

**Response:**

```json
{
  "payload": {
    "depositBalance": 50000.00
  }
}
```

| Field                     | Type   | Description               |
|---------------------------|--------|---------------------------|
| `payload.depositBalance`  | double | Current available balance |

---

## Invoice APIs (v2)

### 8. Create Invoice

Create a new payment invoice for customer payment collection.

```
POST {payriff.url}/v2/invoices
```

**Headers:**

| Header          | Value              |
|-----------------|--------------------|
| `Authorization` | `<PAYRIFF_TOKEN>`  |
| `Content-Type`  | `application/json` |
| `Accept`        | `application/json` |

**Request Body:**

```json
{
  "merchant": "<MERCHANT_ID>",
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

| Field                         | Type               | Description                        |
|-------------------------------|--------------------|------------------------------------|
| `merchant`                    | String             | Merchant ID                        |
| `body.amount`                 | Double             | Invoice amount                     |
| `body.approveURL`             | String             | Redirect URL on approval           |
| `body.cancelURL`              | String             | Redirect URL on cancellation       |
| `body.currencyType`           | String             | Currency code (e.g. AZN, USD)      |
| `body.customMessage`          | String             | Custom message to display          |
| `body.declineURL`             | String             | Redirect URL on decline            |
| `body.description`            | String             | Invoice description                |
| `body.email`                  | String             | Customer email                     |
| `body.expireDate`             | LocalDateTime      | Expiration date/time               |
| `body.fullName`               | String             | Customer full name                 |
| `body.installmentPeriod`      | Integer            | Installment period in months       |
| `body.installmentProductType` | String             | Installment product type           |
| `body.languageType`           | String             | Language (AZ, EN, RU)              |
| `body.phoneNumber`            | String             | Customer phone number              |
| `body.sendSms`                | Boolean            | Send SMS notification              |
| `body.sendWhatsapp`           | Boolean            | Send WhatsApp notification         |
| `body.sendEmail`              | Boolean            | Send email notification            |
| `body.amountDynamic`          | Boolean            | Allow customer to change amount    |
| `body.directPay`              | Boolean            | Direct payment without redirect    |
| `body.metadata`               | Map<String,String> | Custom key-value metadata          |

**Response**: Raw JSON (variable structure from Payriff).

---

### 9. Get Invoice Info

Retrieve invoice details by its UUID.

```
POST {payriff.url}/v2/get-invoice
```

**Headers:**

| Header          | Value              |
|-----------------|--------------------|
| `Authorization` | `<PAYRIFF_TOKEN>`  |
| `Content-Type`  | `application/json` |
| `Accept`        | `application/json` |

**Request Body:**

```json
{
  "merchant": "<MERCHANT_ID>",
  "body": {
    "uuid": "invoice-uuid-here"
  }
}
```

| Field       | Type   | Description   |
|-------------|--------|---------------|
| `merchant`  | String | Merchant ID   |
| `body.uuid` | String | Invoice UUID  |

**Response**: Raw JSON (variable structure from Payriff).

---

## Endpoint Summary

| #  | Method | Payriff Path                           | Version | Description            |
|----|--------|----------------------------------------|---------|------------------------|
| 1  | POST   | `/v3/payout/check-cardholder`          | v3      | Validate cardholder    |
| 2  | POST   | `/v3/payout`                           | v3      | Process payout         |
| 3  | GET    | `/v3/payout/info/{identifier}`         | v3      | Get payout info        |
| 4  | POST   | `/v3/payout/by-search`                 | v3      | Search payouts         |
| 5  | GET    | `/v3/card/check-card-info/{cardBin}`   | v3      | Card info by BIN       |
| 6  | GET    | `/v3/payout/receipt/{rrn}`             | v3      | Download receipt PDF   |
| 7  | GET    | `/v2/deposit`                          | v2      | Check balance          |
| 8  | POST   | `/v2/invoices`                         | v2      | Create invoice         |
| 9  | POST   | `/v2/get-invoice`                      | v2      | Get invoice info       |

---

## Error Handling

Payriff returns standard HTTP error codes. The service handles them as follows:

| HTTP Status | Handling                                                    |
|-------------|-------------------------------------------------------------|
| `2xx`       | Success — response body parsed into typed DTOs              |
| `4xx`       | `HttpClientErrorException` — `message` field extracted from response body |
| `5xx`       | Generic exception wrapped in `PayoutException`              |

---

## Configuration Reference

| Property                  | Description                       | Source |
|---------------------------|-----------------------------------|--------|
| `payriff.url`             | Payriff API base URL              | Vault  |
| `payriffpayment.token`    | Authorization token               | Vault  |
| `payriffpayment.merchantId` | Merchant identifier             | Vault  |
| `payout.bypass.fin-codes` | Comma-separated FIN codes to bypass Payriff calls | Properties |
