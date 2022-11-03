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
    
        
    
    
    
    
# Instalasi menggunakan docker

## get docker image

`docker pull primekey/signserver-ce:5.2.0`

