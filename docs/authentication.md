---
sidebar_position: 4
---

# Authentication

All EcoSwap endpoints require a valid JWT (JSON Web Token) for authentication. The token must be included in the `Authorization` header of every request, using the Bearer scheme:

```
Authorization: Bearer <JWT_ACCESS_TOKEN>
```

A valid token ensures that the request originates from an authorized partner and that the operations performed are permitted. Tokens have limited lifetimes and must be refreshed periodically.

## Obtaining an Access Token

To interact with EcoSwap’s APIs, you must first obtain an access token. EcoSwap will provide you with a `clientId` and `clientSecret`. Using these credentials, you can request a token from the token issuance endpoint.

**Endpoint:**
```
POST https://<host_name>/api/v1/tokens/get
Content-Type: application/x-www-form-urlencoded
```

**Request Parameters (Body):**
- `clientId` (String, Mandatory): The client identifier assigned to you.
- `clientSecret` (String, Mandatory): The secret associated with your client identifier.

<details>
<summary>Example Request & Response</summary>

**Example Request (cURL):**
```bash
curl --location 'https://dev.ecoswapvn.com/api/v1/tokens/get' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'clientId=<YOUR_CLIENT_ID>' \
--data-urlencode 'clientSecret=<YOUR_CLIENT_SECRET>'
```

**Successful Response (HTTP 200):**
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

**Usage Notes:**
- Store the `accessToken` securely and avoid exposing it to unauthorized parties.
- Track the `expireTimeMs` to know when to refresh the token.
- Never embed the `clientSecret` in client-side code or logs.

## Refreshing an Access Token

When your `accessToken` is close to expiring or has expired, you can use the `refreshToken` to obtain a new `accessToken` without reusing your `clientId` and `clientSecret`. This enhances security by reducing the frequency of handling primary credentials.

**Endpoint:**
```
POST https://<host_name>/api/v1/tokens/refresh
Authorization: Bearer <CURRENT_ACCESS_TOKEN>
Content-Type: application/x-www-form-urlencoded
```

**Request Parameters (Body):**
- `refreshToken` (String, Mandatory): The refresh token obtained from the initial token response.

<details>
<summary>Example Request & Response</summary>

**Example Request (cURL):**
```bash
curl --location 'https://dev.ecoswapvn.com/api/v1/tokens/refresh' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Authorization: Bearer <CURRENT_ACCESS_TOKEN>' \
--data-urlencode 'refreshToken=<REFRESH_TOKEN>'
```

**Successful Response (HTTP 200):**
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

**Usage Notes:**
- Replace your old `token` and `refreshToken` with the newly returned ones.
- Always use the latest `refreshToken` to ensure continuous token renewal.
- If the `refreshToken` is invalid or expired, request a new access token using `clientId` and `clientSecret`.

## Error Responses (Token Handling)

When requesting or refreshing tokens, you may encounter errors:

- `00400` **Bad Request**:  
  Missing or invalid parameters.  
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

- `00401` **Unauthorized**:  
  Invalid credentials, expired token, or incorrect `refreshToken`.  
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

If you receive unauthorized errors, verify that your `clientId`, `clientSecret`, and `refreshToken` are correct and current. If necessary, contact support.

## Best Practices

- **Secure Storage:** Keep your `clientId`, `clientSecret`, and tokens secure at all times.
- **Expiration Handling:** Monitor `expireTimeMs` and proactively refresh your token before it expires to avoid service interruptions.
- **Limited Scope of Use:** Only use the obtained tokens for the authorized APIs and do not share them with any unauthorized third parties.
