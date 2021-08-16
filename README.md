# Signserver

using primekey signserver

`docker-compose --project-name signserver -f .\docker-compose.yml up`

`docker-compose --project-name signserver -f .\docker-compose.yml down`

`docker-compose --project-name signserver -f .\docker-compose.yml down --remove-orphans --volumes`


## setting up pki signserver

 - **generate pair key**:   
   menghasilkan file keystore berisi 1 entry private key. entry key dalam keystore ini akan dimount di */mnt/persistent/secrets/tls/ss.gehirn.org/server.jks* dan diexport oleh signserver melalui script kedalam file keystore internalnya pada path */opt/primekey/wildfly-22.0.1.Final/standalone/configuration/keystore.jks*      
  `keytool -genkey -alias server-alias -keyalg RSA -keypass passw0rd -storepass passw0rd -keystore keystore.jks`

 - **export generated certificate**:   
   mengexport certificate ke file server.cer yang nanti akan diimport ke browser pengakses situs signserver, menandakan situs signserver dipercaya oleh browser untuk diakses   
   `keytool -export -alias server-alias -storepass passw0rd -keystore keystore.jks -file server.cer`
 - **Add the certificate to the trust store file**  
   menyimpan certificate ke trusstore file , saat import ke browser bisa dipilih menggunakan *server.cer* seperti point di atas atau melalui truststore yang terbentuk (dalam hal ini cacerts.jks)   
   `keytool -import -v -trustcacerts -alias server-alias -file server.cer  -keypass passw0rd_cacert -storepass passw0rd_cacert -keystore cacerts.jks`
 - **import certificate ke browser**   
   add atau import certificate ini ke browser sehingga browser mengenali signserver. Import disarankan menggunakan *server.cer* karena lebih mudah, sementara jika menggunakan trustore akan diminta password yang diset saat membuat truststore file tersebut. Pada Firefox diimport ke tag 'Servers'

## setting up pki untuk client/browser

akan digunakan openssl sebagai alternatif
 - **generate pair key**   
   key ada di *key.pem* dan certificate di *certificate.pem*   
   `openssl.exe" req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 3650 -out certificate.pem`
 - **Combine your key and certificate in a PKCS#12 (P12) bundle:**   
   `openssl pkcs12 -inkey key.pem -in certificate.pem -export -out certificate.p12`
 - **instal pair key (public private) ke browser firefox**   
   Install certificate.p12 ke browser firefox pada tab 'Your Certificate'
 

**Catatan tambahan, CN pakai link webadmin sesuai rekomendasi link [ini](https://stackoverflow.com/questions/19486200/is-there-a-way-to-test-2-way-ssl-through-browser/19507156)**

## setting cert client:

mount di path : /mnt/external/secrets/tls/cas/*.crt

## enter container

`docker exec -it signserver bash`

truststore dan keystore akan digenerate ada di path `/opt/primekey/wildfly-22.0.1.Final/standalone/configuration`

check menggunakan 

`keytool -list -v -keystore <file>`

misal   
`keytool -list -v -keystore keystore.jks`   
`keytool -list -v -keystore truststore.jks`   

untuk passwd ada di file :   
/opt/primekey/wildfly-22.0.1.Final/standalone/configuration/standalone.xml   
pada tag:   

```xml
<tls>
    <key-stores>
    <key-store name="httpsKS">
        <credential-reference clear-text="JTMzn7Wkftmny1NNyg3s+Oim"/>
        <implementation type="JKS"/>
        <file path="keystore.jks" relative-to="jboss.server.config.dir"/>
    </key-store>
    <key-store name="httpsTS">
        <credential-reference clear-text="7bLhGmxroJt+pDFswJL+Gunk"/>
        <implementation type="JKS"/>
        <file path="truststore.jks" relative-to="jboss.server.config.dir"/>
    </key-store>
...
```


### akses web

http://ss.gehirn.org:8080/signserver/

## Signer

langkah-langkahnya diambil dari halaman [readme docker signserver](https://hub.docker.com/r/primekey/signserver-ce) 

### tes pdf signer 

di tes ini kita ingin menambahkan pdf signer. Worker yg perlu ditambahkan ada dua: *Crypto token* dan *pdfsigner*

#### alternative 1, using tmeplate

##### Add Crypto Token

 - Go to the SignServer Administration Web.
 - On the Workers page, click Add.
 - Click From Template.
 - Select keystore-crypto.properties in the Load From Template list and click Next.
 - Update the following in the configuration:
    - Remove the "#" before "WORKERGENID1.KEYSTOREPASSWORD=foo123".
 - Click Apply to add the Crypto Token.

##### Add Signers

 - On the Workers page, click Add.
 - Click From Template.
 - Select the properties in the Load From Template list for the signer to add, for example, pdfsigner.properties, and click Next.
 - Click Apply to load the configuration and verify that the signer is in state ACTIVE and ready to be used.

##### Test Signing

 - Go to the SignServer Client Web: https://ss.gehirn.org/signserver/clientweb/
 - Under File Upload, specify the Worker Name used, for example, *PDFSigner*.
 - Click Browse to select a PDF file.
 - Click Submit and store the resulting signature file.

## referensi

- https://stackoverflow.com/questions/19486200/is-there-a-way-to-test-2-way-ssl-through-browser/19507156
- https://hub.docker.com/r/primekey/signserver-ce
- https://github.com/primekeydevs/containers
