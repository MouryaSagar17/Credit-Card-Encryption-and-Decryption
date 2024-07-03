# Credit-Card-Encryption-and-Decryption
This is one such project which helps you combine your cloud computing skills also in your cyber security journey. Access control management and cryptography are the key skills for this project.

Author name : Mourya

+ [Introduction](#introduction)
+ [JWE Encryption and Decryption](#jwe-encryption-and-decryption)
#### Introduction <a name="introduction"></a>
 Access control management and cryptography are the key skills for this project.


#### JWE Encryption and Decryption <a name="jwe-encryption-and-decryption"></a>

+ [Introduction](#jwe-introduction)
+ [Configuring the JWE Encryption](#configuring-the-jwe-encryption)
+ [Performing JWE Encryption](#performing-jwe-encryption)
+ [Performing JWE Decryption](#performing-jwe-decryption)
+ [Encrypting Entire Payloads](#encrypting-entire-payloads-jwe)
+ [Decrypting Entire Payloads](#decrypting-entire-payloads-jwe)
+ [Encrypting Payloads with Wildcards](#encrypting-wildcard-payloads-jwe)
+ [Decrypting Payloads with Wildcards](#decrypting-wildcard-payloads-jwe)

##### • Introduction <a name="jwe-introduction"></a>

This library uses [JWE compact serialization](https://datatracker.ietf.org/doc/html/rfc7516#section-7.1) for the encryption of sensitive data.
The core methods responsible for payload encryption and decryption are `encryptPayload` and `decryptPayload` in the `JweEncryption` class.

* `encryptPayload` usage:
```java
String encryptedRequestPayload = JweEncryption.encryptPayload(requestPayload, config);

```

* `decryptPayload` usage:
```java
String responsePayload = JweEncryption.decryptPayload(encryptedResponsePayload, config);
```

##### • Configuring the JWE Encryption <a name="configuring-the-jwe-encryption"></a>
Use the `JweConfigBuilder` to create `JweConfig` instances. Example:
```java
JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withEncryptionCertificate(encryptionCertificate)
    .withDecryptionKey(decryptionKey)
    .withEncryptionPath("$.path.to.foo", "$.path.to.encryptedFoo")
    .withDecryptionPath("$.path.to.encryptedFoo.encryptedValue", "$.path.to.foo")
    .withEncryptedValueFieldName("encryptedValue")
    .build();
```

See also:
* [Service Configurations for Client Encryption Java](https://github.com/Mastercard/client-encryption-java/wiki/Service-Configurations-for-Client-Encryption-Java)

##### • Performing JWE Encryption <a name="performing-jwe-encryption"></a>

Call `JweEncryption.encryptPayload` with a JSON request payload and a `JweConfig` instance.

Example using the configuration [above](#configuring-the-jwe-encryption):
```java
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
```

Output:
```json
{
    "path": {
        "to": {
            "encryptedFoo": {
                "encryptedValue": "eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw"
            }
        }
    }
}
```

##### • Performing JWE Decryption <a name="performing-jwe-decryption"></a>

Call `JweEncryption.decryptPayload` with a JSON response payload and a `JweConfig` instance.

Example using the configuration [above](#configuring-the-jwe-encryption):
```java
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
```

Output:
```json
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
```

##### • Encrypting Entire Payloads <a name="encrypting-entire-payloads-jwe"></a>

Entire payloads can be encrypted using the "$" operator as encryption path:

```java
JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withEncryptionCertificate(encryptionCertificate)
    .withEncryptionPath("$", "$")
    // …
    .build();
```

Example:
```java
String payload = "{" +
    "    \"sensitiveField1\": \"sensitiveValue1\"," +
    "    \"sensitiveField2\": \"sensitiveValue2\"" +
    "}";
String encryptedPayload = JweEncryption.encryptPayload(payload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(encryptedPayload)));
```

Output:
```json
{
    "encryptedValue": "eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw"
}
```

##### • Decrypting Entire Payloads <a name="decrypting-entire-payloads-jwe"></a>

Entire payloads can be decrypted using the "$" operator as decryption path:

```java
JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withDecryptionKey(decryptionKey)
    .withDecryptionPath("$.encryptedValue", "$")
    // …
    .build();
```

Example:
```java
String encryptedPayload = "{" +
    "  \"encryptedValue\": \"eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw\"" +
    "}";
String payload = JweEncryption.decryptPayload(encryptedPayload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(payload)));
```

Output:
```json
{
    "sensitiveField1": "sensitiveValue1",
    "sensitiveField2": "sensitiveValue2"
}
```

##### • Encrypting Payloads with Wildcards <a name="encrypting-wildcard-payloads-jwe"></a>

Wildcards can be encrypted using the "[*]" operator as part of encryption path:

```java
JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withEncryptionCertificate(encryptionCertificate)
    .withEncryptionPath("$.list[*]sensitiveField1", "$.list[*]encryptedField")
    // …
    .build();
```

Example:
```java
String payload = "{ \"list\": [ " +
    "   { \"sensitiveField1\" : \"sensitiveValue1\"}, "+
    "   { \"sensitiveField1\" : \"sensitiveValue2\"} " +
    "]}";
String encryptedPayload = JweEncryption.encryptPayload(payload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(encryptedPayload)));
```

Output:
```json
{
  "list": [
    {"encryptedField": "eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw"},
    {"encryptedField": "eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+asdvarvasdvfdvakmkmm"}
  ]
}
```

##### • Decrypting Payloads with Wildcards <a name="decrypting-wildcard-payloads-jwe"></a>

Wildcards can be decrypted using the "[*]" operator as part of decryption path:

```java
JweConfig config = JweConfigBuilder.aJweEncryptionConfig()
    .withDecryptionKey(decryptionKey)
    .withDecryptionPath("$.list[*]encryptedField", "$.list[*]sensitiveField1")
    // …
    .build();
```

Example:
```java
String encryptedPayload = "{ \"list\": [ " +
        " { \"encryptedField\": \"eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+oPYKZEMTKyYcSIVEgtQw\"}, " +
        " { \"encryptedField\": \"eyJraWQiOiI3NjFiMDAzYzFlYWRlM….Y+asdvarvasdvfdvakmkmm\"} " +
        " ]}";
String payload = JweEncryption.decryptPayload(encryptedPayload, config);
System.out.println(new GsonBuilder().setPrettyPrinting().create().toJson(new JsonParser().parse(payload)));
```

Output:
```json
{
  "list": [
    {"sensitiveField1": "sensitiveValue1"},
    {"sensitiveField2": "sensitiveValue2"}
  ]
}

