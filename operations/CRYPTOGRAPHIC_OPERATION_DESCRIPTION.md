Define a CRYPTOGRAPHIC_OPERATION step in a JSON workflow language that performs encryption or decryption using RSA_OAEP algorithm.

Basic Structure:
{
  "id": string,
  "operation": "CRYPTOGRAPHIC_OPERATION",
  "type": "encrypt" | "decrypt",
  "algorithm": "RSA_OAEP",
  "input": string,
  "key": string
}

Key Features:
- Supports RSA_OAEP encryption and decryption
- For encryption: input is plaintext, key is PEM-formatted RSA public key, output is base64-encoded ciphertext
- For decryption: input is base64-encoded ciphertext, key is PEM-formatted RSA private key, output is plaintext
- Uses SHA-256 for OAEP hashing
- Requires unique operation ID

Parameters:
- type: The operation type, either "encrypt" or "decrypt" (required)
- algorithm: The cryptographic algorithm to use, currently only "RSA_OAEP" is supported (required)
- input: The string to encrypt (plaintext) or decrypt (base64-encoded ciphertext) (required)
- key: PEM-formatted RSA key - public key for encryption, private key for decryption (required)

Examples:

1. Encrypt a string:
{
  "id": "encryptSecret",
  "operation": "CRYPTOGRAPHIC_OPERATION",
  "type": "encrypt",
  "algorithm": "RSA_OAEP",
  "input": "my secret message",
  "key": "-----BEGIN PUBLIC KEY-----\\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A..."
}

Output: Base64-encoded ciphertext string

2. Decrypt a string:
{
  "id": "decryptSecret",
  "operation": "CRYPTOGRAPHIC_OPERATION",
  "type": "decrypt",
  "algorithm": "RSA_OAEP",
  "input": "base64EncodedCiphertext...",
  "key": "-----BEGIN PRIVATE KEY-----\\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASC..."
}

Output: Original plaintext string

Implementation Notes:
- The 'type' field must be either 'encrypt' or 'decrypt'
- The 'algorithm' field must be 'RSA_OAEP' (only supported algorithm currently)
- For encryption, provide the RSA public key in PEM format
- For decryption, provide the RSA private key in PEM format
- The input for decryption must be valid base64-encoded ciphertext
- Output for encryption is base64-encoded to ensure safe string handling
- The operation uses SHA-256 for both the OAEP hash and the MGF1 mask generation function