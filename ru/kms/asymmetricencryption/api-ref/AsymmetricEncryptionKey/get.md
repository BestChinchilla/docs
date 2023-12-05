---
editable: false
sourcePath: en/_api-ref/kms/v1/asymmetricencryption/api-ref/AsymmetricEncryptionKey/get.md
---

# Key Management Service API, REST: AsymmetricEncryptionKey.get
Returns the specified asymmetric KMS key.
 
To get the list of available asymmetric KMS keys, make a [list](/docs/kms/api-ref/SymmetricKey/list) request.
 
## HTTP request {#https-request}
```
GET https://{{ api-host-kms }}/kms/v1/asymmetricEncryptionKeys/{keyId}
```
 
## Path parameters {#path_params}
 
Parameter | Description
--- | ---
keyId | <p>Required. ID of the asymmetric KMS key to return. To get the ID of an asymmetric KMS key use a <a href="/docs/kms/api-ref/AsymmetricEncryptionKey/list">list</a> request.</p> <p>The maximum string length in characters is 50.</p> 
 
## Response {#responses}
**HTTP Code: 200 - OK**

```json 
{
  "id": "string",
  "folderId": "string",
  "createdAt": "string",
  "name": "string",
  "description": "string",
  "labels": "object",
  "status": "string",
  "encryptionAlgorithm": "string",
  "deletionProtection": true
}
```
An asymmetric KMS key that may contain several versions of the cryptographic material.
 
Field | Description
--- | ---
id | **string**<br><p>ID of the key.</p> 
folderId | **string**<br><p>ID of the folder that the key belongs to.</p> 
createdAt | **string** (date-time)<br><p>Time when the key was created.</p> <p>String in <a href="https://www.ietf.org/rfc/rfc3339.txt">RFC3339</a> text format. The range of possible values is from ``0001-01-01T00:00:00Z`` to ``9999-12-31T23:59:59.999999999Z``, i.e. from 0 to 9 digits for fractions of a second.</p> <p>To work with values in this field, use the APIs described in the <a href="https://developers.google.com/protocol-buffers/docs/reference/overview">Protocol Buffers reference</a>. In some languages, built-in datetime utilities do not support nanosecond precision (9 digits).</p> 
name | **string**<br><p>Name of the key.</p> 
description | **string**<br><p>Description of the key.</p> 
labels | **object**<br><p>Custom labels for the key as ``key:value`` pairs. Maximum 64 per key.</p> 
status | **string**<br><p>Current status of the key.</p> <ul> <li>CREATING: The key is being created.</li> <li>ACTIVE: The key is active and can be used for encryption and decryption or signature and verification. Can be set to INACTIVE using the [AsymmetricKeyService.Update] method.</li> <li>INACTIVE: The key is inactive and unusable. Can be set to ACTIVE using the [AsymmetricKeyService.Update] method.</li> </ul> 
encryptionAlgorithm | **string**<br><p>Asymmetric Encryption Algorithm ID.</p> <p>Supported asymmetric encryption algorithms.</p> <ul> <li>RSA_2048_ENC_OAEP_SHA_256: RSA-2048 encryption with OAEP padding and SHA-256</li> <li>RSA_3072_ENC_OAEP_SHA_256: RSA-3072 encryption with OAEP padding and SHA-256</li> <li>RSA_4096_ENC_OAEP_SHA_256: RSA-4096 encryption with OAEP padding and SHA-256</li> </ul> 
deletionProtection | **boolean** (boolean)<br><p>Flag that inhibits deletion of the key</p> 