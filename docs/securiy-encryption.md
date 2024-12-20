---
sidebar_position: 3
---
# Security & Encryption

To ensure the secure transmission of sensitive data, the integration leverages a hybrid encryption approach combining **RSA** and **AES** algorithms. This ensures both the confidentiality and integrity of data exchanged between Samsung and EcoSwap.

## Overview

1. **Asymmetric Encryption (RSA):**  
   Used to securely exchange the AES key. Samsung encrypts a generated AES key with EcoSwap’s public RSA key. Only EcoSwap’s corresponding private RSA key can decrypt this, ensuring no unauthorized parties can intercept the encryption key.

2. **Symmetric Encryption (AES):**  
   Once Samsung obtains the AES key, it uses this symmetric key to encrypt the JSON payload. AES is computationally efficient for encrypting bulk data. The encrypted payload is then Base64-encoded before transmission.

By separating the key exchange (RSA) from the bulk data encryption (AES), the solution provides robust security. If the encrypted payload is intercepted, it cannot be decrypted without the AES key, which is itself protected by RSA encryption.

## Process Flow

1. **Key Generation (Partner Side):**
   - EcoSwap generates a private/public RSA key pair.
   - EcoSwap shares the public key (`public.pem`) with Samsung.
   
   **Example of Key Generation:**
   ```bash
   sudo openssl genrsa -out ./private.pem 2048
   sudo openssl rsa -pubout -in ./private.pem -out ./public.pem
   ```
   
   - **private.pem**: EcoSwap’s private key (keep secret).
   - **public.pem**: EcoSwap’s public key (shared with Samsung).

2. **Encryption Steps (Samsung Side):**
   1. Samsung generates a random AES key.
   2. Samsung encrypts the AES key using EcoSwap’s `public.pem` (RSA encryption).
      - The result is `encryptedKey` (Base64 encoded).
   3. Samsung uses the AES key to encrypt the JSON payload (device details, order info, etc.).
      - The result is `payload` (AES-encrypted and then Base64 encoded).
   4. Samsung sends `encryptedKey` and `payload` to EcoSwap.

3. **Decryption Steps (EcoSwap Side):**
   1. EcoSwap receives `encryptedKey` and `payload`.
   2. EcoSwap RSA-decrypts `encryptedKey` with `private.pem` to retrieve the plain AES key.
   3. EcoSwap uses this AES key to AES-decrypt `payload`, obtaining the original JSON data.
   
   **Pseudocode for Decryption:**
   ```bash
   # Decrypt AES key (Base64 -> RSA decrypt):
   base64 -d encryptedKey.b64 | openssl rsautl -decrypt -inkey private.pem -out aes_key.bin

   # Use the decrypted AES key to AES-decrypt the payload (Base64 -> AES decrypt):
   base64 -d payload.b64 | openssl aes-256-cbc -d -K <hex_aes_key> -iv <iv> -out payload.json
   ```
   
   *Note: Actual commands and parameters (e.g., IV, cipher mode) must be agreed upon in advance between Samsung and EcoSwap.*

## Considerations & Best Practices

- **Key Lengths:**  
  Use RSA keys of at least 2048 bits for secure key exchange and AES-256 for payload encryption.
  
- **Initialization Vector (IV):**  
  Ensure the AES encryption includes a securely generated IV. The IV should be agreed upon or included in the message in a secure manner (e.g., appended to the ciphertext and not encrypted with RSA).

- **Integrity Checks:**  
  Consider using cryptographic signatures or message authentication codes (MACs) to ensure that the data hasn’t been tampered with in transit.

- **Regular Key Rotation:**  
  Periodically rotate your RSA key pairs and AES keys to minimize the impact of any potential key compromise.

- **Error Handling:**  
  If RSA decryption fails or the AES decryption fails, respond with an error. This typically indicates either incorrect keys, corrupted data, or unauthorized tampering.

## Summary

The RSA-AES hybrid encryption ensures that:
- The AES key cannot be obtained by unauthorized parties since it is protected by RSA encryption.
- The payload remains confidential and secure, even if intercepted, without the corresponding AES key.
- Both parties can trust that the information is transmitted securely and privately.
