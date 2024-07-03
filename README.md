# Credit-Card-Encryption-and-Decryption
This is one such project which helps you combine your cloud computing skills also in your cyber security journey. Access control management and cryptography are the key skills for this project.

Author name : Mourya

JWE Encryption and Decryption
Introduction
Configuring the JWE Encryption
Performing JWE Encryption
Performing JWE Decryption
Encrypting Entire Payloads
Decrypting Entire Payloads
Encrypting Payloads with Wildcards
Decrypting Payloads with Wildcards

• Introduction
This library uses JWE compact serialization for the encryption of sensitive data. The core methods responsible for payload encryption and decryption are encryptPayload and decryptPayload in the JweEncryption class.

encryptPayload usage:
String encryptedRequestPayload = JweEncryption.encryptPayload(requestPayload, config);
decryptPayload usage:
String responsePayload = JweEncryption.decryptPayload(encryptedResponsePayload, config);
• Configuring the JWE Encryption
Use the JweConfigBuilder to create JweConfig instances. Example:

JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withEncryptionCertificate(encryptionCertificate)
    .withDecryptionKey(decryptionKey)
    .withEncryptionPath("$.path.to.foo", "$.path.to.encryptedFoo")
    .withDecryptionPath("$.path.to.encryptedFoo.encryptedValue", "$.path.to.foo")
    .withEncryptedValueFieldName("encryptedValue")
    .withIVSize(16) // available values are 12 or 16. If not specified, default value is 16.
    .build();
See also:

Service Configurations for Client Encryption Java
• Performing JWE Encryption
Call JweEncryption.encryptPayload with a JSON request payload and a JweConfig instance.

Example using the configuration above:

String payload = "{" +
    "    \"path\": {" +
    "        \"to\": {" +
    "            \"foo\": {" +
    "                \"sensitiveField1\": \"sensitiveValue1\"," +
    "                \"sensitiveField2\": \"sensitiveValue2\"" +
    "            }" +
    "        }" +
    "    }" +
    "}";
String encryptedPayload = JweEncryption.encryptPayload(payload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(encryptedPayload)));
Output:

{
    "path": {
        "to": {
            "encryptedFoo": {
                "encryptedValue": "eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw"
            }
        }
    }
}
• Performing JWE Decryption
Call JweEncryption.decryptPayload with a JSON response payload and a JweConfig instance.

Example using the configuration above:

String encryptedPayload = "{" +
    "    \"path\": {" +
    "        \"to\": {" +
    "            \"encryptedFoo\": {" +
    "                \"encryptedValue\": \"eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw\"" +
    "            }" +
    "        }" +
    "    }" +
    "}";
String payload = JweEncryption.decryptPayload(encryptedPayload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(payload)));
Output:

{
    "path": {
        "to": {
            "foo": {
                "sensitiveField1": "sensitiveValue1",
                "sensitiveField2": "sensitiveValue2"
            }
        }
    }
}
• Encrypting Entire Payloads
Entire payloads can be encrypted using the "$" operator as encryption path:

JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withEncryptionCertificate(encryptionCertificate)
    .withEncryptionPath("$", "$")
    // …
    .build();
Example:

String payload = "{" +
    "    \"sensitiveField1\": \"sensitiveValue1\"," +
    "    \"sensitiveField2\": \"sensitiveValue2\"" +
    "}";
String encryptedPayload = JweEncryption.encryptPayload(payload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(encryptedPayload)));
Output:

{
    "encryptedValue": "eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw"
}
• Decrypting Entire Payloads
Entire payloads can be decrypted using the "$" operator as decryption path:

JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withDecryptionKey(decryptionKey)
    .withDecryptionPath("$.encryptedValue", "$")
    // …
    .build();
Example:

String encryptedPayload = "{" +
    "  \"encryptedValue\": \"eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw\"" +
    "}";
String payload = JweEncryption.decryptPayload(encryptedPayload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(payload)));
Output:

{
    "sensitiveField1": "sensitiveValue1",
    "sensitiveField2": "sensitiveValue2"
}
• Encrypting Payloads with Wildcards
Wildcards can be encrypted using the "[*]" operator as part of encryption path:

JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withEncryptionCertificate(encryptionCertificate)
    .withEncryptionPath("$.list[*]sensitiveField1", "$.list[*]encryptedField")
    // …
    .build();
Example:

String payload = "{ \"list\": [ " +
    "   { \"sensitiveField1\" : \"sensitiveValue1\"}, "+
    "   { \"sensitiveField1\" : \"sensitiveValue2\"} " +
    "]}";
String encryptedPayload = JweEncryption.encryptPayload(payload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(encryptedPayload)));
Output:

{
  "list": [
    {"encryptedField": "eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw"},
    {"encryptedField": "eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+asdvarvasdvfdvakmkmm"}
  ]
}
• Decrypting Payloads with Wildcards
Wildcards can be decrypted using the "[*]" operator as part of decryption path:

JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withDecryptionKey(decryptionKey)
    .withDecryptionPath("$.list[*]encryptedField", "$.list[*]sensitiveField1")
    // …
    .build();
Example:

String encryptedPayload = "{ \"list\": [ " +
        " { \"encryptedField\": \"eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw\"}, " +
        " { \"encryptedField\": \"eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+asdvarvasdvfdvakmkmm\"} " +
        " ]}";
String payload = JweEncryption.decryptPayload(encryptedPayload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(payload)));
Output:

{
  "list": [
    {"sensitiveField1": "sensitiveValue1"},
    {"sensitiveField2": "sensitiveValue2"}
  ]
}
