---
sidebar_position: 6
---

# Error Handling & Common Codes

This section provides details on how errors are represented and what the common error codes mean. All error responses follow a consistent JSON structure, making it easier to handle them programmatically.

**General Structure of Error Responses:**
```json
{
  "status": {
    "code": "<ERROR_CODE>",
    "desc": "<HUMAN_READABLE_DESCRIPTION>"
  },
  "data": null
}
```

- `code`: A string representing the error or status code.
- `desc`: A short description in English explaining the error.

**Example of a 400 Bad Request Error:**
<details>
<summary>View Example</summary>

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

---

## Common Error Codes

The table below lists common error codes you may encounter during integration, along with their descriptions and scenarios.

| Code   | Description                                                    | Scenario / Possible Causes            |
|--------|----------------------------------------------------------------|---------------------------------------|
| 00000  | Success                                                        | The request completed successfully.   |
| 00400  | Bad Request                                                    | Missing mandatory fields, invalid data formats, or malformed requests. |
| 00401  | Unauthorized                                                   | Invalid or expired token, missing authentication header. |
| 00403  | Forbidden                                                      | Requesting a resource or action not permitted by your credentials. |
| 00404  | Data not found                                                 | The requested resource (e.g., Sales Order ID) does not exist. |
| 09400  | Already bid (Duplicate submit)                                 | A price or bid has already been submitted for this Sales Order. |
| 09401  | Invalid SO State                                               | The requested operation is not allowed in the current Sales Order state. For example, submitting a price when the SO is not in a state that allows it. |
| 00500  | Internal Server Error                                          | An unexpected error occurred on the server. Retry or contact support if persistent. |

---

## Detailed Examples

### 401 Unauthorized Error

This error occurs when the provided Bearer token is missing, invalid, or expired.

<details>
<summary>View Example</summary>

**Response (401 Unauthorized):**
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

**Possible Causes:**
- Not including `Authorization: Bearer <token>` in the header.
- Token has expired.
- Using an incorrect or tampered token.

**Resolution Steps:**
- Check that you have included a valid JWT token.
- Refresh your token if it has expired.
- Ensure the `Authorization` header is spelled and formatted correctly.

---

### 400 Bad Request Error

This error indicates there’s something wrong with the request’s structure or data. Common reasons include missing mandatory fields or invalid parameter formats.

<details>
<summary>View Example</summary>

**Response (400 Bad Request):**
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

**Possible Causes:**
- Missing required fields like `soId` or `event`.
- Invalid data type (e.g., providing a string when a number is expected).
- Unrecognized event type.

**Resolution Steps:**
- Validate your request body against the specification.
- Ensure all mandatory fields are present and of the correct type.
- Double-check JSON syntax and formatting.

---

### 404 Data Not Found Error

This error occurs if you reference a Sales Order or resource that does not exist in the system.

<details>
<summary>View Example</summary>

**Response (404 Data Not Found):**
```json
{
  "status": {
    "code": "00404",
    "desc": "Data not found"
  },
  "data": null
}
```
</details>

**Possible Causes:**
- Passing an incorrect or non-existent `soId`.
- The resource may have been deleted or never created.

**Resolution Steps:**
- Verify the `soId` or resource identifier you are using.
- Confirm that the resource exists by checking logs or retrying with a known valid `soId`.

---

### 500 Internal Server Error

Indicates an unexpected server-side issue. Often transient but can also mean a logic error on the server.

<details>
<summary>View Example</summary>

**Response (500 Internal Server Error):**
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

**Possible Causes:**
- Unexpected exception within the server’s processing logic.
- Temporary service disruptions.

**Resolution Steps:**
- Retry the request after some time.
- If the error persists, contact EcoSwap support with details and request logs.

---

### 09400 Already Bid Error

This occurs if you attempt to submit a price more than once for the same Sales Order.

<details>
<summary>View Example</summary>

**Response (Already Bid 09400):**
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

**Possible Causes:**
- A price submission was previously made and recorded.
  
**Resolution Steps:**
- Check if you already submitted a price for this order.
- If you need to update a previously submitted price, check the allowed operations for that Sales Order state.

---

### 09401 Invalid SO State Error

If you try to perform an action that isn’t allowed in the Sales Order’s current state, you’ll see this error.

<details>
<summary>View Example</summary>

**Response (Invalid SO State 09401):**
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

**Possible Causes:**
- Attempting to submit a price when the SO is not in a state that accepts price submissions (e.g., after it’s already accepted or canceled).

**Resolution Steps:**
- Check the current `soStatus` of the Sales Order before attempting the operation.
- Only perform the action when the SO is in a valid state.

---

**In summary**, the error responses are straightforward and consistent. By using these codes and their descriptions, you can quickly determine what went wrong and take the appropriate corrective action (e.g., fixing request parameters, refreshing tokens, or contacting support).
