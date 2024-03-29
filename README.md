# Signserver

using primekey signserver

`docker-compose --project-name signserver -f .\docker-compose.yml up`

`docker-compose --project-name signserver -f .\docker-compose.yml down`

`docker-compose --project-name signserver -f .\docker-compose.yml down --remove-orphans --volumes`

#### scriptfor testing

`docker run -it --rm --name signserver_test -p 8443:8443 -p 443:8443 -p 8080:8080 -h ss.gehirn.org -v .\certa_pem\certificate.pem:/mnt/external/secrets/tls/cas/ManagementCA1.crt -v .\certa\usingejbca_cert\privet_ssgehirn.key:/mnt/persistent/secrets/tls/ss.gehirn.org/server.key primekey/signserver-ce:5.2.0`

## setting up pki signserver

 - **generate pair key**:   
   menghasilkan file keystore berisi 1 entry private key. entry key dalam keystore ini akan dimount di */mnt/persistent/secrets/tls/ss.gehirn.org/server.jks* dan diexport oleh signserver melalui script kedalam file keystore internalnya pada path */opt/primekey/wildfly-22.0.1.Final/standalone/configuration/keystore.jks*      
  `keytool -genkey -alias ss.gehirn.org -keyalg RSA -keypass passw0rd -storepass passw0rd -keystore keystore.jks`.   
  Untuk CN isikan hostname signserver, mis. ss.gehirn.com.   
  Sample full DN adalah sbb:   
  *CN=ss.gehirn.org, OU=signserver, O=primekey, L=Depok, ST=Jabar, C=ID*

 - **export generated certificate**:   
   mengexport certificate ke file server.cer yang nanti akan diimport ke browser pengakses situs signserver, menandakan situs signserver dipercaya oleh browser untuk diakses   
   `keytool -export -alias ss.gehirn.org -storepass passw0rd -keystore keystore.jks -file server.cer`
 - **Add the certificate to the trust store file (opt.)**  
   menyimpan certificate ke trusstore file , saat import ke browser bisa dipilih menggunakan *server.cer* seperti point di atas atau melalui truststore yang terbentuk (dalam hal ini cacerts.jks)   
   `keytool -import -v -trustcacerts -alias ss.gehirn.org -file server.cer  -keypass passw0rd_cacert -storepass passw0rd_cacert -keystore cacerts_4_java_apps.jks`   
   opsi pada point ini diperlukan untuk apps yg perlu akses sign server certificate melalui mekanisme trust store.   
   Pada aplikasi Java bisa menggunakan opsi JVM `-Djavax.net.ssl.trustStore` dan `-Djavax.net.ssl.trustStorePassword`
 - **import certificate ke browser (chrome)**   
   add atau import certificate ini ke browser sehingga browser mengenali signserver. Import disarankan menggunakan *server.cer* karena lebih mudah, sementara jika menggunakan trustore akan diminta password yang diset saat membuat truststore file tersebut. Import atau simpan pada bagian certificate store: *Trusted Root CA*
 - **import certificate ke browser (Firefox)**
   add atau import melalui akses ke signserver dan add exception. Certificate tersimpan di tab 'Servers' pada *Settings->certificates*

Setelah langkah ini bisa jalankan script signserver (tanpa client truststore) dengan memasang atau mounting 'keystore.jks' ke keystore signserver (via mount ke path */mnt/persistent/secrets/tls/ss.gehirn.org/server.jks* jika menggunakan docker).   
Untuk setting client trustore lakukan langkah berikutnya.

**Catatan tambahan**, CN untuk signserver certificate pakai hostname signserver sesuai rekomendasi link [ini](https://stackoverflow.com/questions/19486200/is-there-a-way-to-test-2-way-ssl-through-browser/19507156)


## setting up pki untuk client/browser

akan digunakan openssl sebagai alternatif
 - **generate pair key**   
   key ada di *key.pem* dan certificate di *certificate.pem*   
   `openssl.exe req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 3650 -out certificate.pem`
 - **Combine your key and certificate in a PKCS#12 (P12) bundle:**   
   `openssl pkcs12 -inkey key.pem -in certificate.pem -export -out certificate.p12`
 - **instal pair key (public private) ke browser (firefox)**   
   Install certificate.p12 ke browser firefox pada tab 'Your Certificate'
 - **instal pair key (public private) ke browser (chrome)**   
   Install certificate.p12 ke browser chrome pada bagian 'Personal'

 
pada projek ini, certificate (public key) yang digunakan oleh browser (dan oleh karena itu ada atau terpasang pada truststore signserver) berada di posisi `../certa_pem/certificate.pem:/mnt/external/secrets/tls/cas/ManagementCA1.crt:ro` di dalam file docker-compose

catatan:   
untuk membedakan dengan certificate lain, berikut contoh full DN untuk certtificate browser:   
*EMAILADDRESS=seele@gmail.com, CN=seele browser, OU=browser, O=browser ltd, L=Depok, ST=Jabar, C=ID*

## setting cert client:

mount di path : `/mnt/external/secrets/tls/cas/*.crt`

## enter container

`docker exec -it signserver bash`

truststore dan keystore akan digenerate ada di path `/opt/primekey/wildfly-22.0.1.Final/standalone/configuration`

atau jika key dan certificates dimounting di path `/mnt` seperti contoh:

```
      - ./certa/keystore.jks:/mnt/persistent/secrets/tls/ss.kexxx_demo.id/server.jks:ro
      - ./certa/keystore.storepasswd:/mnt/persistent/secrets/tls/ss.kexxx_demo.id/server.storepasswd:ro
      - ./certa_pem/certificate.pem:/mnt/external/secrets/tls/cas/ManagementCA1.crt:ro
      
```

 maka pathnya menjadi

`/opt/primekey/appserver/standalone/configuration`
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

#### alternative 1, using template

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

#### alternative 2, Untuk menambahkan custom sign key and certificate untuk MRTOD

##### Add Crypto token
- Pada laman *SignServer Administration Web* tambahkan worker *keystore-crypto.properties*. Load dari template
- Update the following in the configuration:
    - Change "WORKERGENID1.KEYSTORETYPE=PKCS12" to "WORKERGENID1.KEYSTORETYPE=INTERNAL". (Opsi INTERNAL = to use an in-configuration keystore in other words INTERNAL to use a keystore stored in the database (tied to the crypto worker))
      - When Using an Internal Keystore
        The content of the keystore is not part of the regular worker properties. Thus, it is not included when running the dump properties command. It is also removed when removing the crypto worker (or regular worker when using the legacy method to set up crypto tokens). To backup the content of the crypto token, a database backup should be made. The password supplied when activating the token the first time will be used as the keystore password. [referensi](https://download.primekey.se/docs/SignServer-Enterprise/current/KeyStoreCryptoToken.html)

    - Remove the line starting with "WORKERGENID1.KEYSTOREPATH".
- Apply untuk menambakan Crypto token
- Aktifasi Crypto token yang telah dibuat
- Masukkan passwrd baru, jangan sampai lupa karena dibutuhkan saat aktifasi ulang saat signserver direstart atau tetapkan pada property `WORKERGENID1.KEYSTOREPASSWORD=[isi dengan password]` cara ini membuat token "auto-activated"
- setelahnya masuk ke entry crypto token tersebut dan pilih tab *'Crypto Token'* untuk generate key
- click *Generate key*.
- specify a New Key Alias name for the key, for example "mrtdsod_test"
- Click *Generate* and verify that the worker is now in state **ACTIVE**.

##### Add Signer

Seperti langkah yang dijelaskan sebelumnya saat add Signer alternatif 1:   
 - On the *SignServer Workers* page, click *Add*.
 - Click *From Template*.
 - Select the properties in the *Load From Template* list for the signer to add, for example mrtdsodsigner.properties and click Next.
 - edit *WORKERGENID1.NAME* beri nama yang unik misal 'MRTDSODSignerCustom'.
 - edit *WORKERGENID1.CRYPTOTOKEN* set nama Crypto Token yang tadi dibuat
 - Click Apply to load the configuration. The worker is OFFLINE as it needs a key and certificate.
 
##### Generate Keys and Request and Install Certificates

To generate keys for the signer:
 - Select the signer in the Workers list, and click *Renew key*.
 - Under *Renew Keys*, specify the following:
     - Select *Key Algorithm*, for example RSA.
     - Select *Key Specification*, for example 3072.
     - Specify a name for the new key, for example 'mrtdsodsignerkey001'.
     - Click *Generate*.

To generate the CSR for the signer:
 - Select the signer in the Workers list, and click *Generate CSR*.
 - Specify a DN, for example "CN=MrtdsodSigner 0001", and then click Generate.
 - Click *Download* and store the CSR/PKCS#10 file (extension is .p10 file - X905).

Next, bring the CSR to the CA and obtain a certificate in PEM format for it.   

 - go to ca_server (using ejbca in this case)
 - pada [halaman utama dari EJBCA](https://ca.gehirn.org:9443/ejbca/) pilih *Create Certificate from CSR* pada bagian *enroll*.
 - Di bagian Enroll, masukkan username dari entitas yang sudah kita buat sebelumnya di bagian End Entity Profile pada kolom username dan masukkan juga password yang kita buat bersamaan dengan username tersebut pada entry Enrollment code.
 - Upload file CSR yang sudah kita buat di bagian Request File.
 - Pada bagian Result Type, pilih *PEM – full certificate chain*, download pem file

###### troubleshoot ejbca 
jika muncul error saat Create Certificate from CSR seperti ini:   
*Got request with status GENERATED (40), NEW, FAILED or INPROCESS required:*

When using the AuthenticationSession (default) all users have a STATUS. The status lifecycle begins with NEW and ends with REVOKED. Only when the status is NEW, FAILED or INPROCESS is it possible to issue a certificate to a user. After a certificate has been issued, the status is set to GENERATED. This works like a one-time-password scheme. To issue a new certificate to the user his/her status **must be reset** to NEW, FAILED or INPROCESS. This can be done with the Admin GUI or: `bin/ejbca.sh ra setendentitystatus username status` (Status '10' is NEW) example : `./bin/ejbca.sh ra setendentitystatus --username melcior -S 10`. Just enter `bin/ejbca.sh ra setendentitystatus` to see a list of all status codes.

lihat link [ini](https://download.primekey.com/docs/EJBCA-Enterprise/6_5_5/faq.html#errorUserStatus) untuk detailnya

##### install the signer certificates issued by the CA in SignServer

Then, to install the signer certificates issued by the CA in SignServer, do the following:

 - Select the signer in the *SignServer Workers* list, and click *Install Certificates*.
 - Browse for the PEM certificate file and click Add.
 - Click *Install* and confirm that the signer is now listed as *ACTIVE* and ready to be used.


#### Autentikasi WS Client Signer 

##### Client WS WSDL url

akses wsdl url : [https://host:port/signserver/ClientWSService/ClientWS?wsdl](https://host:port/signserver/ClientWSService/ClientWS?wsdl)

example: 

https://ss.gehirn.org:8443/signserver/ClientWSService/ClientWS?wsdl

##### Admin WS WSDL url


akses wsdl for admin WS 

[http://hostname:port/signserver/AdminWSService/AdminWS?wsdl](http://hostname:port/signserver/AdminWSService/AdminWS?wsdl)

example: 

https://ss.gehirn.org:8443/signserver/AdminWSService/AdminWS?wsdl

##### buat certificate untuk WS API client 

Caranya sama dengan generate certificate lainnya dan pastikan CN nya sama dengan host signserver (misal ss.gehirn.org):

 - `keytool -genkey -alias mrtdsodauthclient -keyalg RSA -keypass passw0rd -storepass passw0rd -keystore mrtdauth_keystore.jks`   
 untuk membedakan dengan cert lainnya akan digunakan full DN sbb: *CN=ss.gehirn.org, OU=signserver, O=primekey, L=Depok Timur, ST=Jabar, C=ID*
 - `keytool -export -alias mrtdsodauthclient -storepass passw0rd -keystore mrtdauth_keystore.jks -file mrtdauth_server.cer`   
 certificate ini juga dimounting ke trusstore sign server agar bisa dikenali oleh signserver selain ditambahkan juga pada konfigurasi *Authorization* worker.   
 Pada docker compose file certificate dimount ke `/mnt/external/secrets/tls/cas/*.crt`
 - import juga kedalam truststore client, public key milik signserver untuk mengakses WS Service via HTTPS   
 `keytool -import -v -trustcacerts -alias ss.gehirn.org -file C:\certa\server.cer -keypass passw0rd_cacert -storepass passw0rd_cacert -keystore mrtdauth_cacerts.jks`
 - pada aplikasi java, set JVM Option untuk mengarahkan ke keystore   
 `-Djavax.net.ssl.keyStore=` dan `-Djavax.net.ssl.keyStorePassword=`   
 sementara untuk mengakses wsdl atau mengirim request diarahkan ke truststore yg berisi certificate sign server.   
 `-Djavax.net.ssl.trustStore` dan `-Djavax.net.ssl.trustStorePassword`


Dengan tambahan command untuk export cert based X905 yang akan di load pada konfigurasi  *Add authorized Client* pada worker (lihat poin pembahasan selanjutnya *instal certificate untuk WS client*)
 - `keytool -export -alias mrtdsodauthclient -storepass passw0rd -keystore mrtdauth_keystore.jks -rfc -file mrtdauth_server_x905.cer`


##### instal certificate untuk WS client

 - Pasang certificate untuk Client Certification pada konfigurasi *Authorization* pada Web Admin bagian worker
    - CN certificate will be: *ss.gehirn.org*, gunakan pada bagian konfigurasi *Choose certificate field:* 
 - tambahkan atau ubah konfigurasi `AUTHTYPE` menjadi `CLIENTCERT`
 - Jika menggunakan soapui:
     - klik node project dan konfigurasikan di 'WS-Security Configuration -> Keystores tab'
     - tambahkan certificate yang dipasang public keynya dalam sign server. Catatan: key harus disimpan dalam keystore yang disimpan via perintah seperti: `openssl.exe pkcs12 -inkey key.pem -in certificate.pem -export -out certificate.p12`
     - open request ws di soapui arahkan **Request Properties ** ke **SSL keystore**
 - Jika menggunakan java console apps (java main atau seperti di junit method test)
     - import signserver certificate ke cacerts jdk di path `$java_home\jrlib\security\cacerts`, contoh `C:\jdk1.8.0_202\jre\lib\security\cacerts`
         - `keytool -import -v -trustcacerts -alias ss.gehirn.org -file C:\certa\server.cer -keypass changeit -storepass changeit -keystore cacerts`
     - atau menggunakan opsi -Djavax.net.ssl.keyStore, -Djavax.net.ssl.keyStorePassword, -Djavax.net.ssl.trustStore, -Djavax.net.ssl.trustStorePassword. Lihat catatan untuk java apps akses WS


##### catatan untuk java apps akses WS 

jvm : 
` -Djavax.net.ssl.trustStore=C:\certa\cacerts_4_java_apps.jks -Djavax.net.ssl.trustStorePassword=passw0rd_cacert`   
`-Djavax.net.ssl.keyStorePassword=passw0rd -Djavax.net.ssl.keyStore=C:\tuk_ws\mrtdauth_keystore.jks`


### java apps accessing Signserver Web service interface

both keystore and trustore option must exist 

keystore contains private key of apps and its public key must be put inside signserver truststore file, while truststore must contain public key of signserver

#### environment to set

```
-Djavax.net.ssl.trustStore=/trustore_location/trust_store.jks 
-Djavax.net.ssl.trustStorePassword=password 
-Djavax.net.ssl.keyStorePassword=password 
-Djavax.net.ssl.keyStore=/keystore_location/keystore_for_access_signserver.jks
```

## referensi

- https://stackoverflow.com/questions/19486200/is-there-a-way-to-test-2-way-ssl-through-browser/19507156
- https://hub.docker.com/r/primekey/signserver-ce
- https://github.com/primekeydevs/containers
- https://db-blog.web.cern.ch/blog/luis-rodriguez-fernandez/2014-07-java-soap-client-certificate-authentication
- https://stackoverflow.com/questions/15755293/apache-cxf-wsdl-download-via-ssl-tls
- https://download.primekey.com/docs/EJBCA-Enterprise/6_5_5/faq.html#errorUserStatus

## Rest Microservices

Rest service menggunakan code dari [sini](https://github.com/suryogumilar/RestClientSignServer)

Embedded Microservice as REST routing ke signserver pada file `docker-compose-withmicroservice.yml`

Pada bagian volume untuk service rest-service adalah truststore dan keystore milik rest service (cxf) yang berkomunikasi dengan signserver   
```
  volumes:
      
      - ../certa/cacerts_4_java_apps.jks:/mnt/forss/cacerts_4_java_apps.jks:ro
      - ../dckr_vols/tuk_ws/mrtdauth_keystore.jks:/mnt/forss/mrtdauth_keystore.jks
```

untuk rest service sendiri yang berkomunikasi dengan client yang mengakses rest WS service pada bagian   
```
  volumes:
      - ../apps_folder:/usr/src/app
      - ../dckr_vols/tuk_rest/certificate.p12:/mnt/forrest/certificate.p12:ro
...
      - ../dckr_vols/tuk_rest/truststore_rest_service.jks:/mnt/forrest/truststore_rest_service.jks:ro
```

untuk lebih jelasnya bisa dilihat pada bagian *contoh script* dari README [RestClientSignServer](https://github.com/suryogumilar/RestClientSignServer)


### run all service 

`docker-compose --project-name signserver -f .\docker-compose-withmicroservice.yml up`

`docker-compose --project-name signserver -f .\docker-compose-withmicroservice.yml down`

`docker-compose --project-name signserver -f .\docker-compose-withmicroservice.yml down --remove-orphans --volumes`