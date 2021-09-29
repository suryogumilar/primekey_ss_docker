# Primekey EJBCA

an attempt to implement EJBCA using docker

`docker pull primekey/ejbca-ce:7.4.3.2`


to run:   
`docker run -it --rm --name ca_server -p 9090:8080 -p 9443:8443 -h ca.gehirn.org -e TLS_SETUP_ENABLED="simple" primekey/ejbca-ce:7.4.3.2`

akses ke link   
https://ca.gehirn.org:9443/ejbca/adminweb/

*catatan: jika menggunakan chrome mungkin akan menemui error NET::ERR_CERT_AUTHORITY_INVALID dan setiap akses laman tersebut kadang akan muncul page konfirmasi. Gunakan incognito mode untuk mengaksesnya agar terhindar dari page konfirmasi chrome yang sedikit mengganggu saat edit konfigurasi*


### membuat Certificate

Akses https://ca.gehirn.org:9443/ejbca/adminweb/

 - masuk ke **Certificate Profiles** under *CA Functions*
 - clone profile **Server** dan beri nama Certificate Profiles yang sudah di clone, misal *CSR Server* (CSR = Certificate Signing Request. Pada case ini kita akam membuat server untuk melayani csr). Tekan  *Create From Template*
 - Edit profile *CSR Server*
     - pada *Key Usage* pilih *use* dan *critical*
     - pilih juga opsi *Digital Signature* dan *Key encipherment*
     - pada *Extended key usage* pilih *Use*
         - pilih juga *Server authentication* dan *client authentication* 
     - pada *Authority Information Access* Centang bagian *Use* dan gunakan OCSP Locator yang sudah didefinisikan sebelumnya pada CA (*Use CA defined OCSP locator*) dan tambahan: (*Use CA defined CA issuer*).
     - Entry *Available CAs* set ke *ManagementCA*
     - Save
 - go to *RA Functions* dan ke *End Entity Profiles*
     - tambahkan entry *CSR Profile* (type it and then add) lalu edit: 
       - *End Entity E-mail* di-*untick* pada opsi *Use* 
       - *untick* opsi *modifiable* pada saat isi *CN, Common name*
       - *Default Certificate Profile* ke *CSR Server*
       - *Available Certificate Profiles* dipilih *CSR Server* saja
       - *Default CA* diisi *ManagementCA* begitu juga dengan entry *Available CAs*
       - *Default Token* diisi *User Generated*
       - *Available Tokens* diisi *User Generated* saja
 - Selanjutnya mari kita tambahkan entitas yang nantinya akan *digunakan untuk mengenerate sebuah certificate*. Caranya:
     - Pilih *Add Entity* (atau *Add End Entity* di versi 7.4.3.2).
     - Pada bagian *End Entity Profile* pada laman tsb, Pilih CSR Server.
     - Masukkan Username dan Password sesuai dengan yang kita inginkan (dalam hal ini usernamenya kita tentukan *melcior* - bisa apa saja)
     - Masukkan alamat email jika kita pilih *Use* pada entry *End Entity E-mail* pada langkah sebelumnya.
     - Pada contoh kali ini, untuk Subject DN Attributes, Saya hanya menggunakan CN (Common Name). Isi sesuai dengan FQDN Server yang ingin kita pasangkan Sertifikat SSL (misal ss.gehirn.org).
     - Kalau sudah selesai, tekan tombol Add
 - Save

### Pembuatan CSR (Certificate Signing Request) di Server yang akan dipasang certificate

Buat CSR (Certificate Signing Request). CSR adalah sebuah encoded text yang diberikan kepada sebuah CA (Certification Authority) ketika mengajukan sebuah sertifikat SSL. CSR biasanya berisi informasi yang akan disertakan dalam sertifikat seperti nama organisasi (O), Common Name (nama domain), lokalitas dan negara. CSR juga berisi kunci publik yang akan disertakan dalam sertifikat. Ketika membuat CSR, kita juga akan membuat Private Key.

perintah:   
`openssl req -new -newkey rsa:2048 -nodes -keyout <nama_privatekey> -out <nama_csr>`

contoh:   
`openssl req -new -newkey rsa:2048 -nodes -keyout privet_ssgehirn.key -out csr_ssgehirn.csr`

untuk entry CN, Masukkan FQDN Server: misal: *ss.gehirn.org*

### Generate Certificate

pada CA Server (ejbca) kita upload CSR yang sudah dibuat tadi untuk generate sertifikat SSL. Caranya:

 - pada [halaman utama dari EJBCA](https://ca.gehirn.org:9443/ejbca/) pilih *Create Certificate from CSR* pada bagian *enroll*.
 - Di bagian *Enroll*, masukkan username dari entitas yang sudah kita buat sebelumnya di bagian *End Entity Profile* pada kolom username (*melcior*) dan masukkan juga password yang kita buat bersamaan dengan username tersebut pada entry *Enrollment code*.
 - Upload file CSR yang sudah kita buat di bagian *Request File*.
 - Pada bagian Result Type, pilih *PEM – certificate only*, kemudian tekan OK
 - pindahkan file hasil yang sudah disign (downloaded) ke folder public dan private yg sudah terlebih dahulu dibuat (yang menggunakan openssl tadi). Sehingga di folder ada 3 file: csr, key dan certificate

### Download Certificate Chain 
kita harus mengunduh 1 file lagi yaitu CA chain certificate dari CA yang menerbitkan sertifikat ini

Pada [laman ejbca](https://ca.gehirn.org:9443/ejbca) kita download CA chain certificate.   
Pada bagian *Retrieve*, Pilih *Fetch CA Certificates*. List CA yang tersedia pun akan muncul. Pilih link *Download PEM Chain*.

Kalau sudah di download, masukkan file *chain.pem* (ManagementCA-chain.pem) tersebut ke dalam direktori sertifikat. Kini di dalam folder tersebut sudah ada 4 file.
 - Certificate : ssgehirnorg.pem
 - Key : privet_ssgehirn.key
 - Certificate Chain : ManagementCA-chain.pem
 - CSR : csr_ssgehirn.csr (sudah tidak diperlukan)

simpan key ke dalam keystore *server.jks* tapi terleibh dahulu ubah ke pkcs12 (menyimpan key dan certificate - ingat, A certificate contains a public key.) 

Convert the certificate and private key to PKCS 12:   
`openssl pkcs12 -export -in ssgehirnorg.pem -inkey privet_ssgehirn.key -name ss.gehirn.org -out ssgehirnbundle-PKCS-12.p12`

Import the certificate to the keystore:   
`keytool -importkeystore -deststorepass [password] -destkeystore keystore.jks -srckeystore ssgehirnbundle-PKCS-12.p12 -srcstoretype PKCS12`

Check dengan command:   
`keytool -list -v -keystore keystore.jks`


lalu keystore.jks bisa dipasang di server/ signserver   

certificate CA (CN=ManagementCA) juga dipasang di browser, diambil melalui laman *Fetch CA certificates* pada link [ini](https://ca.gehirn.org:9443/ejbca/retrieve/ca_certs.jsp).   
Impor cert tersebut ke dalam browser (firefox - saat download certificate pakai link *Download to Firefox*). Tahap pertama, Buka browser dan masuk ke dalam bagian *Preferences -> Advanced -> Certificates -> View Certificates*. Pada bagian Authorities, pilih Import dan kemudian upload Root CA Cert yang sudah di-download tadi (file biasanya : ManagementCA.pem). Pilih trust untuk identify websites dan (opsional) identify email users

###### error: SEC_ERROR_INADEQUATE_CERT_TYPE

Kadang muncul error : SEC_ERROR_INADEQUATE_CERT_TYPE coba untick opsi *modifiable* dan pastikan pada *Available Certificate Profiles* di *End Entity Profile* terpilih CSR Server saja

#####  note
dari keytool ketika import dari pkcs ke jks 

The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using:   
`keytool -importkeystore -srckeystore keystore.jks -destkeystore keystore.jks -deststoretype pkcs12`

##### sign server primekey docker container

command untuk run container signserver untuk testing:

`docker run -it --rm --name signserver_test -p 8443:8443 -p 443:8443 -p 8080:8080 -h ss.gehirn.org -v .\certa_pem\certificate.pem:/mnt/external/secrets/tls/cas/ManagementCA1.crt -v .\certa\usingejbca_cert\keystore.jks:/mnt/persistent/secrets/tls/ss.gehirn.org/server.jks -v .\certa\usingejbca_cert\keystore.storepasswd:/mnt/persistent/secrets/tls/ss.gehirn.org/server.storepasswd primekey/signserver-ce:5.2.0`

### referensi tambahan: build image untuk openssl lib and tools

`docker build -t local-openssl:0.0.1 -f Dockerfile.centos_openssl .`

`docker run -it --rm --name openssl_container -v ./usingejbca_cert:/mnt/external local-openssl:0.0.1 bash`

#### referensi 
 - https://blog.goreinnamah.com/blog/2018/02/21/menerbitkan-sertifikat-ssl/
 - https://hub.docker.com/r/primekey/ejbca-ce
 - https://superuser.com/questions/620121/what-is-the-difference-between-a-certificate-and-a-key-with-respect-to-ssl
 - https://www.wowza.com/docs/how-to-import-an-existing-ssl-certificate-and-private-key