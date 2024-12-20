---
sidebar_position: 10
---
# Trade-In Integration API Documentation

## 1. Introduction

This document outlines the integration points between Samsung and EcoSwap for handling trade-in orders. The integration covers three main events:

1. **SO_CREATE**: Notification of a newly created Sales Order.
2. **Submit Price & Grade**: EcoSwap evaluates the device and posts back the proposed price and grading.
3. **SO_RESULT**: Samsung provides the final outcome of the Sales Order to EcoSwap.

**Key Points:**
- Communication is via JSON over HTTPS.
- All payload data from Samsung to EcoSwap is encrypted (RSA + AES).
- EcoSwap APIs require a JWT-based Bearer Token for authentication.
- Multiple environments are available (Development, Staging, Production).

## 2. Environments

| Environment | Base URL                       |
|-------------|--------------------------------|
| Development | `https://dev.ecoswapvn.com`    |
| Staging     | `https://stg.ecoswapvn.com`    |
| Production  | `https://admin.ecoswapvn.com`  |

**Suggested Workflow:**
- Start integration and testing on Development.
- Move to Staging for UAT.
- Go live on Production after successful UAT.

## 3. Security & Encryption

The integration uses a hybrid approach:
- **RSA**: For encrypting/decrypting the AES key.
- **AES**: For encrypting/decrypting the JSON payload.

**Process:**
1. Samsung generates an AES key.
2. Samsung encrypts the AES key with EcoSwap’s public RSA key, sends it as `encryptedKey` (Base64).
3. Samsung AES-encrypts the payload with this AES key, sends it as `payload` (Base64).
4. EcoSwap:
   - Uses its private RSA key (`private.pem`) to decrypt `encryptedKey` and retrieve the AES key.
   - Uses the retrieved AES key to AES-decrypt the `payload`, obtaining plain-text JSON.

**Key Generation (Partner Side):**
```bash
sudo openssl genrsa -out ./private.pem 2048
sudo openssl rsa -pubout -in ./private.pem -out ./public.pem
```

- `private.pem`: Keep private (used by EcoSwap).
- `public.pem`: Provided to Samsung for encrypting the AES key.

## 4. Authentication (JWT)

All EcoSwap endpoints require a valid Bearer Token in the `Authorization` header:

```
Authorization: Bearer <JWT_ACCESS_TOKEN>
```

### Obtaining an Access Token

**Endpoint:**
```
POST https://<host_name>/api/v1/tokens/get
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
- `clientId` (Mandatory)
- `clientSecret` (Mandatory)

<details>
<summary>Example Request & Response</summary>

**Request:**
```bash
curl --location 'https://dev.ecoswapvn.com/api/v1/tokens/get' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'clientId=<YOUR_CLIENT_ID>' \
--data-urlencode 'clientSecret=<YOUR_CLIENT_SECRET>'
```

**Success Response (200):**
```json
{
  "status": {
    "code": "00000",
    "desc": "Success"
  },
  "data": {
    "token": "<ACCESS_TOKEN>",
    "refreshToken": "<REFRESH_TOKEN>",
    "expireTimeMs": 1727275110055,
    "tempAccessToken": null
  }
}
```
</details>

Use `token` in the Authorization header for subsequent requests.

### Refreshing an Access Token

**Endpoint:**
```
POST https://<host_name>/api/v1/tokens/refresh
Authorization: Bearer <CURRENT_ACCESS_TOKEN>
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
- `refreshToken` (Mandatory)

<details>
<summary>Example Request & Response</summary>

**Request:**
```bash
curl --location 'https://dev.ecoswapvn.com/api/v1/tokens/refresh' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Authorization: Bearer <CURRENT_ACCESS_TOKEN>' \
--data-urlencode 'refreshToken=<REFRESH_TOKEN>'
```

**Success Response (200):**
```json
{
  "status": {
    "code": "00000",
    "desc": "Success"
  },
  "data": {
    "token": "<NEW_ACCESS_TOKEN>",
    "refreshToken": "<NEW_REFRESH_TOKEN>",
    "expireTimeMs": 1727275110055,
    "tempAccessToken": null
  }
}
```
</details>

**Error Responses (Token Handling):**

- `00401` Unauthorized: Invalid or expired access token.
  
<details>
<summary>Example 401 Error</summary>

```json
{
  "status": {
    "code": "00401",
    "desc": "Unauthorized"
  },
  "data": null
}
```
</details>

- `00400` Bad Request: Missing required parameters.

<details>
<summary>Example 400 Error</summary>

```json
{
  "status": {
    "code": "00400",
    "desc": "Bad Request"
  },
  "data": null
}
```
</details>

## 5. Events & Data Flows

### 5.1 SO_CREATE (From Samsung to EcoSwap)

**Description:**  
Notifies EcoSwap that a new Sales Order (SO) has been created on Samsung’s side.

**Endpoint (POST):**
```
<BASE_URL>/samsung-api/v1/so/create
```

**Headers:**
- `Authorization: Bearer <ACCESS_TOKEN>`
- `Content-Type: application/json`

**Request Fields:**
| Field Name   | Type   | M/O | Description                 |
|--------------|--------|-----|-----------------------------|
| event        | String | M   | Must be `"SO_CREATE"`       |
| encryptedKey | String | M   | RSA-encrypted AES key (B64) |
| soId         | String | M   | Unique Sales Order ID       |
| payload      | String | M   | AES-encrypted JSON (B64)    |

**Decrypted Payload Fields (Plain JSON):**
- `soType` (M): `pre-tradein` or `tradein`
- `soStatus` (M): `OFFER`, `ACCEPTED`, `CANCEL`
- `staffInfo` (M): `{ "staffCode": "XXX", "staffName": "XXX" }`
- `shopInfo` (M): `{ "shopCode": "C013260741", "shopName": "BANANA (CENTRAL WORLD)" }`
- `offerImage` (M): Image URLs (front, back, etc.)
- `offerVideo` (M): Video URLs
- `offerDevice` (M): `{brand, model, modelCode, storageSize, imei}`
- `deviceStatus` (M): Test results (`pass`, `fail`, `none`)
- `deviceScore` (M): Numeric scores for screen, body, battery, operatorLock, etc.
- `offerId` (M): Offer ID
- `offerRemark` (O): Remarks
- `createBy`, `createDatetime`, `lastChangeBy`, `lastChangeDatetime` (M)

<details>
<summary>Example Request & Decrypted Payload</summary>

**Encrypted Request:**
```json
{
  "event": "SO_CREATE",
  "soId": "SO240923AABBCC",
  "encryptedKey": "Base64EncodedEncryptedKey==",
  "payload": "Base64EncodedEncryptedPayload=="
}
```

**Decrypted Payload:**
```json
{
  "soType": "tradein",
  "soStatus": "OFFER",
  "staffInfo": { "staffCode": "ST12345", "staffName": "John Doe" },
  "shopInfo": { "shopCode": "C013260741", "shopName": "BANANA (CENTRAL WORLD)" },
  "offerImage": {
    "imageFrontSide": "https://example.com/img/front.jpg",
    "imageBackSide": "https://example.com/img/back.jpg"
  },
  "offerVideo": {
    "video1": "https://example.com/video/device_check.mp4"
  },
  "offerDevice": {
    "brand": "Samsung",
    "model": "Galaxy Note 10+",
    "modelCode": "SM-XXXXX",
    "storageSize": "256GB",
    "imei": "123456789012345"
  },
  "deviceStatus": {
    "bluetooth": "pass",
    "wifi": "pass",
    "gps": "pass",
    "gsm": "pass",
    "frontCamera": "pass",
    "backCamera": "pass",
    "rotateScreen": "none"
  },
  "deviceScore": {
    "screen": 0,
    "body": 1,
    "battery": 1,
    "operatorLock": 1
  },
  "offerId": "OFFER1234",
  "offerRemark": "Minor scratches",
  "createBy": "user_x",
  "createDatetime": "2024-12-20T10:00:00Z",
  "lastChangeBy": "user_x",
  "lastChangeDatetime": "2024-12-20T10:00:00Z"
}
```
</details>

**Success Response (200):**
```json
{
  "status": "received",
  "message": "Sales order has been recorded."
}
```

**Error Responses:**
- `00400` Bad Request (e.g., missing `soId`)
  
<details>
<summary>Example 400 Error</summary>

```json
{
  "status": {
    "code": "00400",
    "desc": "Bad Request"
  },
  "data": null
}
```
</details>

- `00401` Unauthorized (invalid token)
  
<details>
<summary>Example 401 Error</summary>

```json
{
  "status": {
    "code": "00401",
    "desc": "Unauthorized"
  },
  "data": null
}
```
</details>

- `00500` Internal Server Error
  
<details>
<summary>Example 500 Error</summary>

```json
{
  "status": {
    "code": "00500",
    "desc": "Internal Server Error"
  },
  "data": null
}
```
</details>

### 5.2 Submit Price & Grade (EcoSwap to Samsung)

**Description:**  
After evaluating the device, EcoSwap submits the price and grade to Samsung.

**Endpoint (POST - Samsung Provided):**
```
POST https://<samsung_host>/api/v1/ext/sale-orders/{soId}/submitprice
```

**Headers:**
- `Authorization: Bearer <ACCESS_TOKEN>`
- `Content-Type: application/json`

**Request Fields:**
| Field Name    | Type   | M/O | Description                          |
|---------------|--------|-----|--------------------------------------|
| soId (path)   | String | M   | Sales Order ID (e.g. `SO240513WCCIVC`)|
| evaluateGrade | String | O   | A, B, C, D, F                        |
| evaluatePrice | Number | M   | Integer 100–50,000                   |
| attachImage   | String | O   | Base64-encoded image                 |
| comment       | String | O   | Remarks on the device condition      |

<details>
<summary>Example Request & Success Response</summary>

**Request:**
```bash
curl --location 'https://<samsung_host>/api/v1/ext/sale-orders/SO240513WCCIVC/submitprice' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <ACCESS_TOKEN>' \
--data '{
  "comment": "Great condition, slight scratch on screen corner",
  "evaluateGrade": "A",
  "evaluatePrice": "8000",
  "attachImage": "dGVzdGltYWdlcGF5bG9hZA=="
}'
```

**Success Response (200):**
```json
{
  "status": {
    "code": "00000",
    "desc": "Success"
  },
  "data": {
    "soId": "SO240513WCCIVC"
  }
}
```
</details>

**Error Responses:**
- `09400` Already bid (Duplicate submit):
  
<details>
<summary>Example 09400 Error</summary>

```json
{
  "status": {
    "code": "09400",
    "desc": "Already bid"
  },
  "data": null
}
```
</details>

- `09401` Invalid SO State:
  
<details>
<summary>Example 09401 Error</summary>

```json
{
  "status": {
    "code": "09401",
    "desc": "Invalid SO State"
  },
  "data": null
}
```
</details>

- Other errors: `00400`, `00401`, `00403`, `00404`, `00500` as above.

### 5.3 SO_RESULT (From Samsung to EcoSwap)

**Description:**  
Samsung sends the final outcome of the Sales Order after the bidding or acceptance process.

**Endpoint (POST):**
```
<BASE_URL>/samsung-api/v1/so/result
```

**Headers:**
- `Authorization: Bearer <ACCESS_TOKEN>`
- `Content-Type: application/json`

**Request Fields:**
| Field Name   | Type   | M/O | Description                          |
|--------------|--------|-----|--------------------------------------|
| event        | String | M   | `"SO_RESULT"`                        |
| encryptedKey | String | M   | RSA-encrypted AES key (B64)          |
| soId         | String | M   | Sales Order ID                       |
| payload      | String | M   | AES-encrypted JSON (B64)             |

**Decrypted Payload Fields:**
- `biddingResult` (M): `WIN` or `LOSE`
- `soStatus` (M): `ACCEPTED` or `CANCEL`
- `customerProfile` (O): If `biddingResult=WIN` & `soStatus=ACCEPTED`
  - `name` (M)
  - `mobileNo` (M)
  - `email` (O)
  - `idCardImg` (M)

<details>
<summary>Example Request & Success Response</summary>

**Encrypted Request:**
```json
{
  "event": "SO_RESULT",
  "soId": "SO240923AABBCC",
  "encryptedKey": "Base64EncodedEncryptedKey==",
  "payload": "Base64EncodedEncryptedPayload=="
}
```

**Decrypted Payload:**
```json
{
  "biddingResult": "WIN",
  "soStatus": "ACCEPTED",
  "customerProfile": {
    "name": "Jane Smith",
    "mobileNo": "0812345678",
    "email": "jane.smith@example.com",
    "idCardImg": "https://example.com/idcard.jpg"
  }
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Final status has been recorded."
}
```
</details>

**Error Responses:**
- `00400` Bad Request, `00401` Unauthorized, `00500` Internal Server Error as shown above.

## 6. Common Error Codes

| Code  | Description                          |
|-------|--------------------------------------|
| 00000 | Success                              |
| 00400 | Bad Request (invalid input)           |
| 00401 | Unauthorized (invalid token)          |
| 00403 | Forbidden (no permission)             |
| 00404 | Data not found                       |
| 09400 | Already bid (duplicate submit)        |
| 09401 | Invalid SO State for this action      |
| 00500 | Internal Server Error (unexpected)    |

## 7. Testing & Validation

**Recommended Steps:**
1. **Development:** Test token handling, encryption/decryption, and event flows.
2. **Staging:** Perform UAT with realistic scenarios.
3. **Production:** Deploy after successful UAT and confirmation.

**Test Scenarios:**
- Token retrieval and refresh.
- Correct AES/RSA decryption for events.
- Handling all event types (SO_CREATE, submitprice, SO_RESULT).
- Error handling for all applicable scenarios.

## 8. Contact & Support

For inquiries or support:

- **Email:** support@ecoswapvn.com  
- **Phone:** +1 (555) 123-4567  
- **Support Hours:** Monday–Friday, 9 AM–6 PM (GMT+7)