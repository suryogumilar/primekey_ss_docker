# Openssl cheatsheets (also keytool)

https://www.digicert.com/kb/ssl-support/openssl-quick-reference-guide.htm

## list pkcs12 content using keytool

`keytool -list -v -keystore keystore.p12 -storetype PKCS12`

## PKCS#12 to PEM

`openssl pkcs12 -in keystore.p12 -nocerts -out yourdomain.key -nodes`

`openssl pkcs12 -in keystore.p12 -nokeys -clcerts -out yourdomain.crt`

## Viewing Certificate Information

`openssl x509 -text -in yourdomain.crt -noout`

`openssl x509 -noout -text -nameopt multiline,utf8 -in certificate.pem`

## viewing jks keystore entry

`keytool -v -list -keystore  keystore.jks`

## viewing key.pem (private key)

`openssl rsa -in key.pem -check`
