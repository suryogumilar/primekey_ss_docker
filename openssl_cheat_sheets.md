# Openssl cheatsheets

https://www.digicert.com/kb/ssl-support/openssl-quick-reference-guide.htm

## PKCS#12 to PEM

`openssl pkcs12 -in keystore.p12 -nocerts -out yourdomain.key -nodes`

`openssl pkcs12 -in keystore.p12 -nokeys -clcerts -out yourdomain.crt`

## Viewing Certificate Information

`openssl x509 -text -in yourdomain.crt -noout`