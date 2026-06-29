---
title: "API Testing Payloads"
description: "Ready-to-use request payloads for testing the Verifow AML/CFT API endpoints."
---

# API Testing Payloads

This guide provides copy-paste request bodies for testing common integration scenarios against the Verifow sandbox. All payloads assume the base URL `/api/v1/` and bearer token authentication.

---

## 1. Authentication

### 1.1 Login

`POST /api/v1/auth/login`

```json
{
  "email": "compliance@yourbank.com",
  "password": "YourSecurePassword"
}
```

---

## 2. KYC Verification

### 2.1 Submit Individual KYC Application

`POST /api/v1/kyc/applications`

```json
{
  "entityType": "INDIVIDUAL",
  "firstName": "Chinedu",
  "lastName": "Obi",
  "bvn": "22012345678",
  "nin": "12345678901",
  "phone": "+2348012345678",
  "email": "chinedu.obi@email.com",
  "dateOfBirth": "1990-03-15",
  "tier": "TIER_2"
}
```

---

## 3. Transaction Screening

### 3.1 Screen Transaction — Individual (APPROVE)

`POST /api/v1/transactions/screen`

```json
{
  "externalId": "TXN-TEST-001",
  "type": "TRANSFER",
  "channel": "MOBILE_APP",
  "amount": 50000,
  "currency": "NGN",
  "senderAccountNumber": "0123456789",
  "senderBvn": "22012345678",
  "senderKycStatus": "VERIFIED",
  "senderKycTier": "TIER_2",
  "senderKycVerifiedAt": "2026-05-01T10:00:00Z",
  "senderKycExternalRef": "CORE-BANK-KYC-78432",
  "senderName": "Chinedu Obi",
  "senderBankCode": "058",
  "receiverAccountNumber": "9876543210",
  "receiverName": "Jane Smith",
  "receiverBankCode": "033",
  "receiverCountry": "NG",
  "narration": "Payment for goods",
  "timestamp": "2026-05-08T12:00:00Z",
  "metadata": {
    "entityType": "INDIVIDUAL"
  }
}
```

### 3.2 Screen Transaction — Individual (KYC BLOCK)

`POST /api/v1/transactions/screen`

```json
{
  "externalId": "TXN-TEST-002",
  "type": "TRANSFER",
  "channel": "MOBILE_APP",
  "amount": 5000000,
  "currency": "NGN",
  "senderAccountNumber": "0123456789",
  "senderBvn": "22012345678",
  "senderKycStatus": "PENDING",
  "senderKycTier": "TIER_1",
  "senderKycVerifiedAt": "2026-05-01T10:00:00Z",
  "senderKycExternalRef": "CORE-BANK-KYC-78432",
  "senderName": "John Doe",
  "senderBankCode": "058",
  "receiverAccountNumber": "9876543210",
  "receiverName": "Jane Smith",
  "receiverBankCode": "033",
  "receiverCountry": "NG",
  "narration": "Payment for goods",
  "timestamp": "2026-05-08T12:00:00Z",
  "metadata": {
    "entityType": "INDIVIDUAL"
  }
}
```

### 3.3 Screen Transaction — Sanctions Hit

`POST /api/v1/transactions/screen`

```json
{
  "externalId": "TXN-TEST-003",
  "type": "TRANSFER",
  "channel": "INTERNET_BANKING",
  "amount": 10000000,
  "currency": "NGN",
  "senderAccountNumber": "0123456789",
  "senderBvn": "22012345678",
  "senderKycStatus": "VERIFIED",
  "senderKycTier": "TIER_3",
  "senderKycVerifiedAt": "2026-05-01T10:00:00Z",
  "senderKycExternalRef": "CORE-BANK-KYC-78432",
  "senderName": "John Doe",
  "senderBankCode": "058",
  "receiverAccountNumber": "9876543210",
  "receiverName": "Jane Smith",
  "receiverBankCode": "033",
  "receiverCountry": "NG",
  "narration": "High value transfer",
  "timestamp": "2026-05-08T12:00:00Z",
  "metadata": {
    "entityType": "INDIVIDUAL"
  }
}
```

### 3.4 Screen Transaction — Business / KYB

For corporate senders, your payload must flag the transaction as a business transaction and supply the company's CAC registration number. When your bank has already performed KYB verification outside of Verifow, assert `senderKybStatus: "VERIFIED"` to bypass the Verifow KYB lookup. You should still send the sender's KYC fields, because authorized signatories and directors may still be evaluated under individual KYC rules.

`POST /api/v1/transactions/screen`

```json
{
  "externalId": "TXN-TEST-BIZ-001",
  "type": "TRANSFER",
  "channel": "INTERNET_BANKING",
  "amount": 25000000,
  "currency": "NGN",
  "senderAccountNumber": "0123456789",
  "senderBvn": "22012345678",
  "senderName": "Acme Nigeria Ltd",
  "senderBankCode": "058",
  "senderKycStatus": "VERIFIED",
  "senderKycTier": "TIER_3",
  "senderKycVerifiedAt": "2026-05-01T10:00:00Z",
  "senderKycExternalRef": "CORE-BANK-KYC-78432",
  "senderKybStatus": "VERIFIED",
  "receiverAccountNumber": "9876543210",
  "receiverName": "Jane Smith",
  "receiverBankCode": "033",
  "receiverCountry": "NG",
  "narration": "Invoice payment",
  "timestamp": "2026-05-08T12:00:00Z",
  "metadata": {
    "entityType": "BUSINESS",
    "senderRcNumber": "RC123456"
  }
}
```

**Business screening payload fields**

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `senderKycStatus` | `string` | ✅ | `VERIFIED`, `PENDING`, `IN_PROGRESS`, `REJECTED`, `EXPIRED`, `NONE`. Asserts the sender's individual KYC status. Required in EXTERNAL mode. |
| `senderKycTier` | `string` | ✅ | `TIER_1`, `TIER_2`, `TIER_3`. Required in EXTERNAL mode; enables CBN tier-based limits. |
| `senderKycVerifiedAt` | `string` | | ISO 8601 timestamp of the last KYC verification. |
| `senderKycExternalRef` | `string` | | Internal KYC reference for audit trail (e.g. `CORE-BANK-KYC-78432`). |
| `senderKybStatus` | `string` | ✅ | `VERIFIED`, `PENDING`, `IN_PROGRESS`, `REJECTED`, `EXPIRED`, `NONE`. Asserts the corporate KYB status. Send `VERIFIED` when your bank has independently verified the business. |
| `metadata.entityType` | `string` | ✅ | Must be `"BUSINESS"` to trigger KYB screening. |
| `metadata.senderRcNumber` | `string` | ✅ | CAC registration number (e.g. `"RC123456"`). |

> **Note:** If `senderKybStatus` is omitted for a business transaction, Verifow will attempt a local KYB lookup using `metadata.senderRcNumber`. If no KYB record exists, the transaction will be scored as high risk.

---

## 4. KYB Verification

### 4.1 Submit KYB Application

`POST /api/v1/kyc/kyb/applications`

```json
{
  "companyName": "Acme Nigeria Ltd",
  "rcNumber": "RC123456",
  "tin": "12345678-0001",
  "address": "15 Broad Street, Lagos",
  "incorporationDate": "2015-06-12",
  "directors": [
    { "name": "John Doe", "bvn": "22012345678", "email": "john@acme.ng" }
  ],
  "beneficialOwners": [
    { "name": "Jane Smith", "percentage": 75, "bvn": "22087654321" }
  ]
}
```
