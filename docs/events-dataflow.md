---
sidebar_position: 5
---

# Events & Data Flows

This section describes the main events involved in the integration and how data moves between Samsung and EcoSwap. Each event follows a secure, encrypted communication pattern.

## Overview of Events

1. **SO_CREATE**: Samsung → EcoSwap  
   Samsung notifies EcoSwap when a new Sales Order is created. EcoSwap decrypts, stores, and acknowledges the order.

2. **Submit Price & Grade**: EcoSwap → Samsung  
   After receiving SO_CREATE data, EcoSwap evaluates the device and submits a proposed trade-in price and grade back to Samsung.

3. **SO_RESULT**: Samsung → EcoSwap  
   Samsung sends the final outcome of the Sales Order (e.g., accepted, canceled, win/loss status) after EcoSwap’s price submission.

---

## 1. SO_CREATE (From Samsung to EcoSwap)

**Description:**  
Samsung sends a notification when a new Sales Order is created. The payload is encrypted using the RSA+AES combination. EcoSwap must decrypt and store the order details.

**Endpoint (POST):**
```
<BASE_URL>/samsung-api/v1/so/create
```

**Required Headers:**
- `Authorization: Bearer <ACCESS_TOKEN>`
- `Content-Type: application/json`

**Request Fields:**
| Field Name   | Type   | Mandatory | Description                               |
|--------------|--------|-----------|-------------------------------------------|
| event        | String | Yes       | Must be `"SO_CREATE"`                     |
| encryptedKey | String | Yes       | RSA-encrypted AES key (Base64)            |
| soId         | String | Yes       | Unique Sales Order ID                     |
| payload      | String | Yes       | AES-encrypted JSON payload (Base64)       |

**Decrypted Payload Fields:**
- **soType** (String, M): `pre-tradein` or `tradein`
- **soStatus** (String, M): `OFFER`, `ACCEPTED`, `CANCEL`
- **staffInfo** (Object, M): `{staffCode (String, M), staffName (String, M)}`
- **shopInfo** (Object, M): `{shopCode (String, M), shopName (String, M)}`
- **offerImage** (Object, M): Contains URLs to device images (e.g., `imageFrontSide`, `imageBackSide`, etc.)
- **offerVideo** (Object, M): Contains URLs to device videos (e.g., `video1`, `video2`)
- **offerDevice** (Object, M): `{brand (String), model (String), modelCode (String), storageSize (String), imei (String)}`
- **deviceStatus** (Object, M): Tests results for device components (`pass`, `fail`, or `none`) for fields like `bluetooth`, `wifi`, `gps`, `frontCamera`, etc.
- **deviceScore** (Object, M): Numeric scores indicating condition of screen, body, battery, operatorLock, etc.
- **offerId** (String, M): Offer ID
- **offerRemark** (String, O): Additional remarks
- **createBy**, **createDatetime**, **lastChangeBy**, **lastChangeDatetime** (Strings, M): Metadata fields

<details>
<summary>Encrypted Request Example</summary>

```json
{
  "event": "SO_CREATE",
  "soId": "SO240923AABBCC",
  "encryptedKey": "Base64EncodedEncryptedKey==",
  "payload": "Base64EncodedEncryptedPayload=="
}
```
</details>

<details>
<summary>Decrypted Payload Example</summary>

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

**Possible Error Responses:**
- `00400`: Bad Request (e.g. missing `soId`)
- `00401`: Unauthorized (invalid token)
- `00500`: Internal Server Error

---

## 2. Submit Price & Grade (From EcoSwap to Samsung)

**Description:**  
After receiving and decrypting the SO_CREATE event, EcoSwap evaluates the device and determines a suitable trade-in price and grade. EcoSwap then **calls Samsung’s endpoint** to submit these details.

**Endpoint (POST - Samsung Provided):**
```
POST https://<samsung_host>/api/v1/ext/sale-orders/{soId}/submitprice
```

**Required Headers:**
- `Authorization: Bearer <ACCESS_TOKEN>`
- `Content-Type: application/json`

**Request Fields:**
| Field Name    | Type   | Mandatory | Description                                   |
|---------------|--------|-----------|-----------------------------------------------|
| soId (path)   | String | Yes       | The unique Sales Order ID (e.g., `SO240513WCCIVC`) |
| evaluateGrade | String | No        | Device grade: A, B, C, D, or F                 |
| evaluatePrice | Number | Yes       | Proposed price (integer 100–50,000)            |
| attachImage   | String | No        | Base64 encoded image if available             |
| comment       | String | No        | Additional remarks or evaluation comments      |

<details>
<summary>Example Request</summary>

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
</details>

<details>
<summary>Success Response (200)</summary>

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

**Possible Error Responses:**
- `09400`: Already bid (Price already submitted)
- `09401`: Invalid SO State (Cannot submit price at the current order state)
- Other standard errors: `00400` (Bad Request), `00401` (Unauthorized), `00403` (Forbidden), `00404` (Not Found), `00500` (Internal Server Error)

<details>
<summary>Example Error Responses</summary>

**Already Bid (09400):**
```json
{
  "status": {
    "code": "09400",
    "desc": "Already bid"
  },
  "data": null
}
```

**Invalid SO State (09401):**
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

---

## 3. SO_RESULT (From Samsung to EcoSwap)

**Description:**  
Once Samsung finalizes the status of the Sales Order (e.g., the customer accepts the price, or the order is canceled), Samsung sends a final result notification to EcoSwap.

**Endpoint (POST):**
```
<BASE_URL>/samsung-api/v1/so/result
```

**Required Headers:**
- `Authorization: Bearer <ACCESS_TOKEN>`
- `Content-Type: application/json`

**Request Fields:**
| Field Name   | Type   | Mandatory | Description                            |
|--------------|--------|-----------|----------------------------------------|
| event        | String | Yes       | Must be `"SO_RESULT"`                  |
| encryptedKey | String | Yes       | RSA-encrypted AES key (Base64)         |
| soId         | String | Yes       | Unique Sales Order ID                  |
| payload      | String | Yes       | AES-encrypted JSON payload (Base64)    |

**Decrypted Payload Fields:**
- **biddingResult** (String, M): `WIN` or `LOSE`
- **soStatus** (String, M): `ACCEPTED` or `CANCEL`
- **customerProfile** (Object, O): Only present if `biddingResult=WIN` and `soStatus=ACCEPTED`
  - `name` (String, M)
  - `mobileNo` (String, M)
  - `email` (String, O)
  - `idCardImg` (String, M): URL to the customer's ID card image

<details>
<summary>Encrypted Request Example</summary>

```json
{
  "event": "SO_RESULT",
  "soId": "SO240923AABBCC",
  "encryptedKey": "Base64EncodedEncryptedKey==",
  "payload": "Base64EncodedEncryptedPayload=="
}
```
</details>

<details>
<summary>Decrypted Payload Example</summary>

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
</details>

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Final status has been recorded."
}
```

**Possible Error Responses:**
- `00400`: Bad Request (invalid fields)
- `00401`: Unauthorized (invalid token)
- `00500`: Internal Server Error

---

## Summary of Data Flow

1. **SO_CREATE (Samsung → EcoSwap)**:  
   Samsung → Encrypted Data → EcoSwap  
   EcoSwap decrypts and stores order details → Responds with acknowledgment.

2. **Submit Price & Grade (EcoSwap → Samsung)**:  
   EcoSwap evaluates device → Calls Samsung’s endpoint with price and grade.  
   Samsung responds with success or error codes.

3. **SO_RESULT (Samsung → EcoSwap)**:  
   Samsung sends final order result → Encrypted data → EcoSwap decrypts and updates records.  
   EcoSwap responds with success or error codes.

This secure, bidirectional communication ensures both parties stay synchronized on the status and evaluation of trade-in orders.
