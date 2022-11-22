# Openssl cheatsheets (also keytool)

https://www.digicert.com/kb/ssl-support/openssl-quick-reference-guide.htm

## list pkcs12 content using keytool

`keytool -list -v -keystore keystore.p12 -storetype PKCS12`

## Converting a Java Keystore Into PEM Format


from https://www.baeldung.com/java-keystore-convert-to-pem-format

Java KeyStores are stored in the JKS file format. It's a proprietary format that is specifically for use in Java programs. PKCS#12 KeyStores are non-proprietary and are increasing in popularity — from Java 9 onward, PKCS#12 is used as the default KeyStore format over JKS.

PEM files are also certificate containers — they encode binary data using Base64, which allows the content to be transmitted more easily through different systems.


### JKS to PKCS#12

```
keytool -importkeystore -srckeystore keystore.jks \
   -destkeystore keystore.p12 \
   -srcstoretype jks \
   -deststoretype pkcs12
```


### PKCS#12 to PEM

`openssl pkcs12 -in keystore.p12 -out keystore.pem`

`openssl pkcs12 -in keystore.p12 -nocerts -out yourdomain.key -nodes`

`openssl pkcs12 -in keystore.p12 -nokeys -clcerts -out yourdomain.crt`


### export a single public key certificate out of a JKS and into PEM format using keytool alone

```
keytool -exportcert -alias first-key-pair -keystore keystore.jks -rfc -file first-key-pair-cert.pem
```
## creating keystore and import .crt

`keytool -import -alias testalias -file test.crt -keypass keypass -keystore test.jks -storepass test@123`

## importing .crt into java keystore

`keytool -trustcacerts -keystore "/jdk/jre/lib/security/cacerts" -storepass changeit -importcert -alias testalias -file "/opt/ssl/test.crt"`

## Viewing Certificate Information

`openssl x509 -text -in yourdomain.crt -noout`

`openssl x509 -noout -text -nameopt multiline,utf8 -in certificate.pem`

## viewing jks keystore entry

`keytool -v -list -keystore  keystore.jks`

## viewing key.pem (private key)

`openssl rsa -in key.pem -check`
