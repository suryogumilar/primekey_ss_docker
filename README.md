# Signserver

using primekey signserver

`docker-compose --project-name signserver -f .\docker-compose.yml up`

`docker-compose --project-name signserver -f .\docker-compose.yml down --remove-orphans --volumes`


## setting up pki

 - **generate pair key**:   
   menghasilkan file keystore berisi 1 entry private key    
  `keytool -genkey -alias server-alias -keyalg RSA -keypass passw0rd -storepass passw0rd -keystore keystore.jks`

 - **export generated certificate**:   
   mengexport certificate ke file server.cer yang nanti akan diimport ke browser pengakses situs signserver   
   `keytool -export -alias server-alias -storepass passw0rd -keystore keystore.jks -file server.cer`
 - **Add the certificate to the trust store file**  
   menyimpan certificate ke trusstore file , saat import ke browser bisa dipilih menggunakan *server.cer* seperti point di atas atau melalui truststore   
   `keytool -import -v -trustcacerts -alias server-alias -file server.cer  -keypass passw0rd_cacert -storepass passw0rd_cacert -keystore cacerts.jks`
