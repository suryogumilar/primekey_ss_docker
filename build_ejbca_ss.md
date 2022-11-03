# Membangun infrastruktur untuk EJBCA dan Signserver

## Setting ejbca untuk melakukan digital signing certificate

### Get image and run

`docker pull primekey/ejbca-ce:7.4.3.2`

```
docker run -itd --name ca_server_demo -p 9090:8080 -p 9443:8443 -h ca.imigrasi_demo.id -e TLS_SETUP_ENABLED="simple" primekey/ejbca-ce:7.4.3.2
```

### Akses dan konfigurasi

akses ke link (using incognito chrome):
[https://ca.imigrasi_demo.id:9443/ejbca/adminweb/](https://ca.imigrasi_demo.id:9443/ejbca/adminweb/)


#### membuat Certificate Profile

Akses [https://ca.imigrasi_demo.id:9443/ejbca/adminweb/](https://ca.imigrasi_demo.id:9443/ejbca/adminweb/)

 * masuk ke **Certificate Profiles** untuk *Manage Certificate Profiles* di bawah bagian **CA Functions**. [link](https://ca.imigrasi_demo.id:9443/ejbca/adminweb/ca/editcertificateprofiles/editcertificateprofiles.xhtml)
 * clone profile **SERVER** yang ada di tabel *List of Certificate Profiles* dan beri nama Certificate Profiles yang sudah di clone tersebut, misal *CSR Imigrasi Server* (CSR = Certificate Signing Request. Pada case ini kita akam membuat server untuk melayani csr). Tekan *Create From Template*
 * Edit profile *CSR Imigrasi Server* dengan mengklik tombol **Edit** pada baris tersebut:
    * pada **Key Usage** di bawah *X.509v3 extensions* dan *usages* pilih opsi: **use** dan **critical**. 
       * masih pada bagian yang sama pilih juga opsi **Digital Signature** dan **Key encipherment**
    * pada bagian **Extended key usage** pilih **Use**
       * pilih juga opsi **Server authentication** dan **client authentication**
    * pada **Authority Information Access** Centang bagian **Use** dan gunakan OCSP Locator yang sudah didefinisikan sebelumnya pada CA (**Use CA defined OCSP locator**) dan tambahan pilih juga **Use CA defined CA issuer**.
    * pada bagian **Available CAs** di bawah **Other data** set ke **ManagementCA**
    * klik **Save**
 * go to **RA Functions** dan ke **End Entity Profiles** untuk mengkonfigurasi RA. [Link](https://ca.imigrasi_demo.id:9443/ejbca/adminweb/ra/editendentityprofiles/editendentityprofiles.xhtml)
    * tambahkan entry *CSR Imigrasi Profile* (type it and then klik **add Profile**) lalu pilih entry *CSR Imigrasi Profile* yang sudah ditambahkan tersebut dan edit dengan klik **Edit end entity profile**:
       * Opsi **Use** pada bagian **End Entity E-mail** di-untick 
       * untick opsi **modifiable** pada saat isi text field **CN, Common name**. CN diisi *ss.kemenlu_demo.id* sesuai hostname Server yang ingin kita pasangkan Sertifikat SSL. Jadi setiap *end entity profiles* dipetakan ke satu hostname server. Oleh sebab itu setiap request defaultnya hanya 1 kali ). Bagian ini juga kita membuat FQDN server yang akan dipasangkan sertifikat dengan *Add* atau *Select for removal*
          * setidaknya tambahkan O=Kemenlu dan OU=epassport
       * Pada bagian **Main Certificate Data** set **Default Certificate Profile** ke *CSR Imigrasi Server* dan pada **Available Certificate Profiles** dipilih *CSR Imigrasi Server* saja
       * **Default CA** pada bagian **Main Certificate Data** diisi *ManagementCA* begitu juga dengan entry **Available CAs**
       * **Default Token** pada bagian **Main Certificate Data** diisi *User Generated*
       * begitu juga **Available Tokens** yang ada di bawahnya diisi *User Generated* saja
       * karena ini demo dan supaya bisa dipakai berkali-kali signing, pada **Other data**, entri **Number of allowed requests ** pilih *Use* dan kita coba set 5
    * klik **save**       
 * Selanjutnya mari kita tambahkan entitas yang nantinya akan digunakan untuk mengenerate sebuah certificate (RA). Caranya, masih pada **RA Functions**:
    * Pilih **Add Entity** (atau **Add End Entity** di versi 7.4.3.2 yaitu yang dipakai di dokumen ini). [Link](https://ca.imigrasi_demo.id:9443/ejbca/adminweb/ra/addendentity.jsp)
    * Pada bagian **End Entity Profile** pada laman tsb, Pilih *CSR Imigrasi Profile*.    
    * Masukkan Username dan Password sesuai dengan yang kita inginkan (dalam hal ini usernamenya kita tentukan *melcior* - bisa apa saja)
    * Masukkan alamat email jika kita pilih **Use** pada entry **End Entity E-mail** pada langkah sebelumnya.
    * Pada contoh kali ini, untuk Subject DN Attributes, Saya hanya menggunakan CN (Common Name). Isi sesuai dengan FQDN Server yang ingin kita pasangkan Sertifikat SSL (dalam hal ini adalah *ss.kemenlu_demo.id*).
    * Kalau sudah selesai, tekan tombol **Add**
    
        
## Pembuatan CSR (Certificate Signing Request) di Sign Server yang akan dipasang certificate    

Buat CSR (Certificate Signing Request). CSR adalah sebuah encoded text yang diberikan kepada sebuah CA (Certification Authority) ketika mengajukan sebuah sertifikat SSL. CSR biasanya berisi informasi yang akan disertakan dalam sertifikat seperti nama organisasi (O), Common Name (nama domain), lokalitas dan negara. CSR juga berisi kunci publik yang akan disertakan dalam sertifikat. Ketika membuat CSR, kita juga akan membuat Private Key.    


`openssl req -new -newkey rsa:2048 -nodes -keyout privet_kemludemo.key -out csr_kemludemo.csr`
    
## Generate Certificate

pada CA Server (ejbca) kita upload CSR yang sudah dibuat tadi untuk generate sertifikat SSL. Caranya:

 * pada halaman utama dari EJBCA [https://ca.imigrasi_demo.id:9443/ejbca/](https://ca.imigrasi_demo.id:9443/ejbca/) pilih **Create Certificate from CSR** pada bagian **Enroll**.
 * Di bagian Enroll, masukkan username dari entitas yang sudah kita buat sebelumnya di bagian End Entity Profile pada kolom username (melcior) dan masukkan juga password yang kita buat bersamaan dengan username tersebut pada entry Enrollment code.
 * Upload file CSR yang sudah kita buat di bagian Request File.
 * Pada bagian Result Type, pilih **PEM – certificate only**, kemudian tekan OK
 * pindahkan file hasil yang sudah disign (downloaded) ke folder public dan private yg sudah terlebih dahulu dibuat (yang menggunakan openssl tadi). Sehingga di folder ada 3 file: csr, key dan certificate. Certificate akan bernama `sskemenlu_demoid.pem`    

## Download Certificate Chain

kita harus mengunduh 1 file lagi yaitu CA chain certificate dari CA yang menerbitkan sertifikat ini

Pada laman ejbca [https://ca.imigrasi_demo.id:9443/ejbca/](https://ca.imigrasi_demo.id:9443/ejbca/) kita download CA chain certificate.
Pada bagian **Retrieve**, Pilih **Fetch CA Certificates**. List CA yang tersedia pun akan muncul pada bagian
 
```
CA: ManagementCA
UID=c-0c8oa2agyp0c97v24,CN=ManagementCA,O=EJBCA Container Quickstart
```

Pilih link **Download PEM Chain**.

Kalau sudah di download, masukkan file chain.pem (*ManagementCA-chain.pem*) tersebut ke dalam direktori sertifikat. Kini di dalam folder tersebut sudah ada 4 file.

 * Certificate : sskemenlu_demoid.pem
 * Key : privet_kemludemo.key
 * Certificate Chain : ManagementCA-chain.pem
 * CSR : csr_kemludemo.csr (sudah tidak diperlukan)

simpan key ke dalam keystore server.jks tapi terleibh dahulu ubah ke pkcs12 (menyimpan key dan certificate - ingat, A certificate contains a public key.)

#### Convert the certificate and private key to PKCS 12:

```
openssl pkcs12 -export -in sskemenlu_demoid.pem -inkey privet_kemludemo.key -name ss.kemenlu_demo.id -out sskemenludemobundle-PKCS-12.p12
```

#### Import the certificate to the keystore:

```
keytool -importkeystore -deststorepass foo123 -destkeystore keystore.jks -srckeystore sskemenludemobundle-PKCS-12.p12 -srcstoretype PKCS12
```

Check dengan command untuk memastikan validitas keystore:

`keytool -list -v -keystore keystore.jks`

# Signserver Instalasi menggunakan docker

## get docker image

`docker pull primekey/signserver-ce:5.2.0`


## docker compose


```
version: '3'

services:
  signserver-service:
    container_name: signserver_kemenlu_demo
    image: primekey/signserver-ce:5.2.0
    ports:
      - 8443:8443
      - 443:8443
      - 8080:8080
    hostname: ss.kemenlu_demo.id
    volumes:
      - ./certa/keystore.jks:/mnt/persistent/secrets/tls/ss.kemenlu_demo.id/server.jks:ro
      - ./certa/keystore.storepasswd:/mnt/persistent/secrets/tls/ss.kemenlu_demo.id/server.storepasswd:ro
      - ./certa_pem/certificate.pem:/mnt/external/secrets/tls/cas/ManagementCA1.crt:ro                 

```

## run

`docker-compose --project-name signserver_kemenlu_demo -f ./docker-compose.yml up`


akses via [http://ss.kemenlu_demo.id:8080/signserver](http://ss.kemenlu_demo.id:8080/signserver)

## Add Crypto token worker

 * Pada laman SignServer Administration Web tambahkan worker **keystore-crypto.properties** via pilihan Load dari template
 * Update the following in the configuration:
    * Change **WORKERGENID1.KEYSTORETYPE=PKCS12** to **WORKERGENID1.KEYSTORETYPE=INTERNAL**
    * Remove or comment the line starting with **WORKERGENID1.KEYSTOREPATH**
    * tambahkan baris **WORKERGENID1.KEYSTOREPASSWORD=[isi dengan password]**
    * Apply untuk menambakan Crypto token
 * Aktifasi Crypto token yang telah dibuat. 
 * Masukkan passwrd yg didefinisikan dari **WORKERGENID1.KEYSTOREPASSWORD**. Pada tahap ini crypto worker tetap offline karena key belum digenerate
 * setelahnya masuk ke entry crypto token tersebut dan pilih tab **Crypto Token** untuk generate key
 * click **Generate key**.
 * specify a New Key Alias name for the key, for example "mrtdsod_test"
 * verify that the worker is now in state **ACTIVE**.

## Add mrtdsod Signer

 * On the SignServer Workers page, click **Add**.
 * Click From Template.
 * Select the properties in the Load From Template list for the signer to add, for example **mrtdsodsigner.properties** and click Next.
 * edit WORKERGENID1.NAME beri nama yang unik misal 'MRTDSODSigner'.
 * edit WORKERGENID1.CRYPTOTOKEN set nama Crypto Token yang tadi dibuat
 * uncomment baris **WORKERGENID1.DEFAULTKEY=signer00003**
 * Click Apply to load the configuration. The worker is OFFLINE as it needs a key and certificate.
 
### Generate Keys and Request and Install Certificates

#### generate Key

 * Select the signer in the Workers list, and click **Renew key**.
 * Under Renew Keys, specify the following:
    * Select Key Algorithm, for example RSA.
    * Select Key Specification, for example 3072.
    * Specify a name for the new key, for example 'mrtdsodsignerkey001'.
    * Click Generate. 
    
#### generate the CSR for the signer

 * Select the signer in the Workers list, and click Generate CSR.
 * Specify a DN, for example "CN=MrtdsodSigner 0001", and then click **Generate**.
 * Click **Download** and store the CSR/PKCS#10 file (extension is .p10 file - X905).

#### Bring the CSR to the CA and obtain a certificate in PEM format for it.

 * go to ca_server (using ejbca in this case)
 * pada halaman utama dari EJBCA pilih **Create Certificate from CSR** pada bagian enroll.
 * Di bagian Enroll, masukkan username dari entitas yang sudah kita buat sebelumnya di bagian End Entity Profile pada kolom username dan masukkan juga password yang kita buat bersamaan dengan username tersebut pada entry Enrollment code.
 * Upload file CSR yang sudah kita buat di bagian Request File.
 * Pada bagian **Result Type**, pilih **PEM – full certificate chain**, lalu download pem file

#### install the signer certificates issued by the CA in SignServer

 * Select the signer in the SignServer Workers list, and click **Install Certificates**.
 * Browse for the PEM certificate file and click **Add**.
 * Click **Install** and confirm that the signer is now listed as ACTIVE and ready to be used.


## Add pdf Signer   

 * On the SignServer Workers page, click **Add**.
 * Click From Template.
 * Select the properties in the Load From Template list for the signer to add, for example **pdfsigner.properties** and click **Next**.
 * edit WORKERGENID1.NAME beri nama yang unik misal 'PDFSigner'. 
 * edit WORKERGENID1.CRYPTOTOKEN set nama Crypto Token yang tadi dibuat
 * uncomment baris WORKERGENID1.DEFAULTKEY=signer00003
Click Apply to load the configuration. The worker is OFFLINE as it needs a key and certificate.

## Generate Keys and Request and Install Certificates

### generate Key

* Select the signer in the Workers list, and click **Renew key**.
 * Under Renew Keys, specify the following:
    * Select Key Algorithm, for example RSA.
    * Select Key Specification, for example 3072.
    * Specify a name for the new key, for example 'pdfsignerkey001'.
    * Click Generate.

### generate the CSR for the signer

 * Select the signer in the Workers list, and click Generate CSR.
 * Specify a DN, for example "CN=PdfSigner 0001", and then click **Generate**.
 * Click Download and store the CSR/PKCS#10 file (extension is .p10 file - X905). 
 
### Bring the CSR to the CA and obtain a certificate in PEM format for it.

exact step same as above on part with the same name

### install the signer certificates issued by the CA in SignServer

exact step same as above on part with the same name

   