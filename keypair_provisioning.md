# Steps for TLS keypair provisioning 

These are the steps for keypair provisioning as TLS configuration for signserver/wildfly. 

These consist of generate a keypair, signing its certificate to a CA through CSR
and install it in a keystore that to be used on TLS Configuration (or other configuration that needs it)

## Setting the keystore

### generate private key and its CSR

Command (using `openssl`): 

`openssl req -new -newkey rsa:3072 -nodes -keyout private_ss.key -out csr_ss.csr`

you will be asked to input some information for building a Distinguished Name or a DN that will be saved in CSR files

Some of the fields are :

 - Country Name (2 letter code)
 - State or Province Name (full name)
 - Locality Name (eg, city)
 - Organization Name (eg, company)
 - Organizational Unit Name (eg, section)
 - Common Name (e.g. server FQDN or YOUR name)
 - Email Address
 - 'extra' attributes to be sent with your certificate request:
   - A challenge password
   - An optional company name

We could also generate CSR from existing private key

`openssl req -out CSR_tmp.csr -key private_ss.key -new`

the content inside CSR_tmp.csr and csr_ss.csr would be the same if you input the same Distinguished Name (DN)

### Send CSR to be signed by CA

there are some online tools for testing purpose in regards of CA's CSR signing for example signed by [getaCert](https://getacert.com/signacert.html)

you will received signed public certificate either in .cer or .pem format also getaCert's public certificate/key(.cer) for Root/signing CA.

Download it

### Convert the certificate and private key to PKCS 12:

`openssl pkcs12 -export -in signed_certificate.cer -inkey private_ss.key -name signserver -out keystore.p12`

then enter export password

### Import the certificate to the keystore using `keytool`

Although Signserver kan read p12 but as alternative you can save it as jks

`keytool -importkeystore -deststorepass foo123 -destkeystore keystore.jks -srckeystore keystore.p12 -srcstoretype PKCS12`

use the keystore as keystore for signserver or wildfly


## Other Operations

#### Verify CSR using openssl

Decode csr using : 

`openssl req -in csr_ss.csr -noout -text` 

#### Verify Private key using openssl

Check a private key:

`openssl rsa -in private_ss.key -check`

Decode private key using openssl:

`openssl rsa -in private_ss.key -noout -text`

`-noout` specifies that an encoded version of the private key should not be included in output.

Decrypt an RSA Private Key Using OpenSSL:

`openssl rsa -in private_ss.key  -out decrypted_private.key`

Check X509 certificate

`openssl x509 -in signed_certificate.cer -text -noout`


list p12

`openssl pkcs12 -info -in keystore.p12`

remember when you have to fill challenge password when creating private key? you have to fill it (Enter PEM pass phrase) again in order to view the private key inside the keystore

``

### Glossary

**RSA(Rivest-Shamir-Adleman)** is an Asymmetric encryption technique that uses two different keys as public and private keys to perform the encryption and decryption. With RSA, you can encrypt sensitive information with a public key and a matching private key is used to decrypt the encrypted message.

**PKCS #8** is a standard syntax for storing private key information. PKCS #8 private keys are typically exchanged in the PEM base64-encoded format, for example:

```
-----BEGIN PRIVATE KEY-----
MIIBVgIBADANBgkqhkiG9w0BAQEFA....
...
srDVjIT3LsvTqw==
-----END PRIVATE KEY-----
```

**X.509** Public and Private keys are encoded differently. Whilst private keys are encoded in PKCS #8, public keys are not. They are instead encoded in X.509 according to the ASN.1 specifications.


**.cer** Certificate with .cer extension usually begins and ends with :

```
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
```

while

**.pem** usually begins with

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 3072 (0xc00)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=US, ....

```

### Troubleshoot

**ERR_CERT_COMMON_NAME_INVALID**: means browser failed to verify website SSL Certificate. Usually happens when common name is not equal to the website's domain name.

**ERR_CERT_AUTHORITY_INVALID**: means browser cannot verify this certificate because it is not in the Trusted Root Certification Authorities store of the browser or OS or using self signed certificate.

### reference

 - https://www.javatpoint.com/rsa-encryption-algorithm
 - https://www.devglan.com/online-tools/rsa-encryption-decryption#:~:text=RSA(Rivest%2DShamir%2DAdleman,to%20decrypt%20the%20encrypted%20message.
 - [Some list of openssl commands](https://gist.github.com/Hakky54/b30418b25215ad7d18f978bc0b448d81) for check and verify your keys
