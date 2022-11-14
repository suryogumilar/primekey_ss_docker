# Konfigurasi worker
## using PKCS12

```

CRYPTOTOKEN=CryptoTokenP12
KEYSTORETYPE=PKCS12

  ALLOW_DETACHEDSIGNATURE_OVERRIDE=FALSE

  DEFAULTKEY=mydefaultkey

  SIGNERCERT=

  KEYSTOREPATH=/opt/keys/dss10_signer2.p12

  AUTHTYPE=NOAUTH

  DETACHEDSIGNATURE=FALSE

  CLASSPATH=org.signserver.common.ProcessableConfig

  SIGNATUREALGORITHM=SHA1withRSA

  SIGNERCERTCHAIN=

  NAME=CMSSigner
```

## using pkcs11

```
Status of CryptoWorker with ID 1 (CryptoTokenP11-tsa) is:
   Worker status : Active
   Token status  : Active

   Worker properties:
      ATTRIBUTES=attributes(generate,CKO_PUBLIC_KEY,*) = {
         CKA_TOKEN = false
         CKA_ENCRYPT = false
         CKA_VERIFY = true
         CKA_WRAP = false
      }
      attributes(generate, CKO_PRIVATE_KEY,*) = {
         CKA_TOKEN = true
         CKA_PRIVATE = true
         CKA_SENSITIVE = true
         CKA_EXTRACTABLE = false
         CKA_DECRYPT = false
         CKA_SIGN = true
         CKA_UNWRAP = false
      }
      
      SLOTLABELTYPE=SLOT_NUMBER
      
      IMPLEMENTATION_CLASS=org.signserver.server.signers.CryptoWorker
      
      DEFAULTKEY=testKey
      
      SHAREDLIBRARYNAME=Utimaco
      
      SLOTLABELVALUE=0
      
      PIN=●●●●●●
      
      TYPE=CRYPTO_WORKER
      
      CRYPTOTOKEN_IMPLEMENTATION_CLASS=org.signserver.server.cryptotokens.PKCS11CryptoToken
      
      NAME=CryptoTokenP11-tsa
      

```