# Instalasi signserver

instalasi manual. menggunakan image centos8.3.2011

## Membuat network container

`docker network create seele_network # Create the network`

## container

`docker run -itd -v C:\insta:/mnt/insta -p 28080:8080 -p 28443:8443 -p 28442:8442 -p 29990:9990 --name insta_ss --net seele_network centos:centos8.3.2011`

`docker exec -it insta_ss bash`

### port 

Port 8080 akan digunakan untuk traffic HTTP umum, port 8442 akan digunakan untuk HTTPS dengan autentikasi server, dan port 8443 untuk HTTPS dengan autentikasi klien dan server. Sementara 9990 akan digunakan sebagai port administrasi wildfly.

## prerequisite

 - OpenJDK 8 (Java)
 - Wildfly 10+ (Application Server)
 - Mariadb 10+ (Database)
 - MariaDB Java Client 2.1.0 
 - Apache Ant
 - Apache Maven 3 (Opsional)
 - Signserver 5.2.0 

### package pre-instalasi

ensuring your system is up-to-date.

```
dnf --assumeyes install epel-release
dnf --assumeyes update
```

instal script service jika belum ada

`dnf --assumeyes install initscripts`

instal net-tools untuk cek network

`dnf --assumeyes install net-tools`

install nano untuk edit script

`dnf --assumeyes install nano`

install unzip kalau belum ada

`dnf --assumeyes install unzip`

instal wget

`dnf --assumeyes install wget`

## instalasi

### instal open JDK 8

`dnf --assumeyes install java-1.8.0-openjdk-devel`

lalu test JAVA_HOME, set jika belum

`echo $JAVA_HOME`

`nano /etc/profile.d/java.sh`

tulis dalam file java.sh tersebut: 

```
JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el8_4.x86_64"
export JAVA_HOME
```

### instal Apache Ant

move ke /usr/local/ant

```
mkdir -p /usr/local/ant
cd /usr/local/ant
```

download ant

`wget https://dlcdn.apache.org//ant/binaries/apache-ant-1.10.11-bin.zip`

unzip apache ant

`unzip apache-ant-1.10.11-bin.zip`

set ANT environment ke /etc/profile.d

`nano /etc/profile.d/ant.sh`

```
ANT_HOME="/usr/local/ant/apache-ant-1.10.11"
PATH="$PATH:/usr/local/ant/apache-ant-1.10.11/bin"
export ANT_HOME
export PATH
```

### install JBoss Wildfly

Tentang JBoss Wildfly:   
WildFly, formerly known as JBoss AS, or simply JBoss, is an application server authored by JBoss, now developed by Red Hat. WildFly is written in Java and implements the Java Platform, Enterprise Edition (Java EE) specification. It runs on multiple platforms.

#### instalasi 

instal menggunakan script `wildfly.sh` akan menginstal di direktori `/opt/wildfly`. Ubah script sesuaikan dengan versi Wildfly yang digunakan.

```sh
WILDFLY_VERSION=10.1.0.Final
WILDFLY_FILENAME=wildfly-$WILDFLY_VERSION
WILDFLY_ARCHIVE_NAME=$WILDFLY_FILENAME.tar.gz
WILDFLY_DOWNLOAD_ADDRESS=https://download.jboss.org/wildfly/$WILDFLY_VERSION/$WILDFLY_ARCHIVE_NAME
```

execute the script

`./wildfly.sh`

check service wildfly

`service wildfly status`

`service --status-all ## check semua service`

start wildfly

`service wildfly start`

isi dari /etc/default/wildfly   

```
JBOSS_HOME="/opt/wildfly"
JBOSS_USER=wildfly
WILDFLY_HOME="/opt/wildfly"
WILDFLY_USER="wildfly"
STARTUP_WAIT=240
SHUTDOWN_WAIT=30
WILDFLY_CONFIG=standalone.xml
WILDFLY_MODE=standalone
WILDFLY_BIND=0.0.0.0
```

lokasi script service ada di : `/etc/systemd/system/wildfly.service`

###### docker problm

since we are using docker, mungkin kita ga bisa up service wildfly untuk 'cheat' kita pakai script set environment:

`nano /etc/profile.d/our_wildfly.sh`

isinya

```
JBOSS_HOME="/opt/wildfly"
JBOSS_USER=wildfly
WILDFLY_HOME="/opt/wildfly"
WILDFLY_USER="wildfly"
STARTUP_WAIT=240
SHUTDOWN_WAIT=30
WILDFLY_CONFIG=standalone.xml
WILDFLY_MODE=standalone
WILDFLY_BIND=0.0.0.0
LAUNCH_JBOSS_IN_BACKGROUND=1

export JBOSS_HOME
export JBOSS_USER
export WILDFLY_HOME
export WILDFLY_USER
export STARTUP_WAIT
export SHUTDOWN_WAIT
export WILDFLY_CONFIG
export WILDFLY_MODE
export WILDFLY_BIND
export LAUNCH_JBOSS_IN_BACKGROUND

export APPSRV_HOME=/opt/wildfly
export PATH=$PATH:$JBOSS_HOME/bin
```

sebagai user **wildfly** start melaui command : 

`docker exec -it -u wildfly insta_ss bash`

`/opt/wildfly/bin/launch.sh $WILDFLY_MODE $WILDFLY_CONFIG $WILDFLY_BIND`

##### tes if wildfly is running

open browser buka http://localhost:8080

uji untuk masuk ke halaman admin dari Wildfly: http://localhost:9990 

#### Konfigurasi Web Server Keystore

Kopikan keystore dan trustore yang telah dibuat ke path:

`WILDFLY_HOME/standalone/configuration/keystore/keystore.jks`

`WILDFLY_HOME/standalone/configuration/keystore/truststore.jks`


as user wildfly :

```
mkdir -p  $WILDFLY_HOME/standalone/configuration/keystore 
cp keystore.jks $WILDFLY_HOME/standalone/configuration/keystore/keystore.jks
cp keystore.storepasswd $WILDFLY_HOME/standalone/configuration/keystore/keystore.storepasswd
```

add user certificate to trustore:

```
keytool -import -v -trustcacerts -alias ss.gehirn.org -file certificate.pem -keypass passw0rd -storepass passw0rd -keystore $WILDFLY_HOME/standalone/configuration/keystore/truststore.jks

nano $WILDFLY_HOME/standalone/configuration/keystore/truststore.storepasswd
```

## Konfigurasi TLS dan HTTP

 - Jalankan JBoss CLI dengan perintah:
   ```
   jboss-cli.sh
   connect
   ```
 - Untuk menghapus konfigurasi TLS dan HTTP yang sudah ada dan mengizinkan port 8443, jalankan perintah berikut:   
   ```
   /subsystem=undertow/server=default-server/http-listener=default:remove 
   /subsystem=undertow/server=default-server/https-listener=https:remove 
   /socket-binding-group=standard-sockets/socket-binding=http:remove
   /socket-binding-group=standard-sockets/socket-binding=https:remove 
   :reload
   ```   
   setelah oeprasi ini, port http(s) tidak bisa diakses, it's okay :)
   
 - Konfigurasi interface dengan menggunakan bind address 0.0.0.0   
   ```
   /interface=http:add(inet-address="0.0.0.0")
   /interface=httpspub:add(inet-address="0.0.0.0")
   /interface=httpspriv:add(inet-address="0.0.0.0")
   ```
 - Konfigurasi HTTPS *httpspriv listener* dan atur agar port privat tersebut membutuhkan sertifikat klien. Gunakan nilai yang tepat untuk key-alias (hostname), password (keystore-password), ca-certificate-password (trustore password), dan protokol yang didukung. Untuk WIldfly 14, gunakan enable-http2=”false” untuk menghindari pesan error dalam log   
   ```
   /core-service=management/security-realm=SSLRealm:add()
   /core-service=management/security-realm=SSLRealm/server-identity=ssl:add(keystore-path="keystore/keystore.jks", keystore-relative-to="jboss.server.config.dir", keystore-password="passw0rd", alias="ss.gehirn.org")
   :reload

   /core-service=management/security-realm=SSLRealm/authentication=truststore:add(keystore-path="keystore/truststore.jks", keystore-relative-to="jboss.server.config.dir", keystore-password="passw0rd")
   :reload

   /socket-binding-group=standard-sockets/socket-binding=httpspriv:add(port="8443",interface="httpspriv")
   /subsystem=undertow/server=default-server/https-listener=httpspriv:add(socket-binding="httpspriv", security-realm="SSLRealm", verify-client=REQUIRED, max-post-size="10485760", enable-http2="true") 
   ```

 - Configure the default HTTP listener.
For WildFly 14, instead use enable-http2="false" to avoid error messages in the log. Perintah ini untuk mengaktifkan request http via port 8080.  
   ```
   /socket-binding-group=standard-sockets/socket-binding=http:add(port="8080",interface="http")
   /subsystem=undertow/server=default-server/http-listener=default:add(socket-binding=http, max-post-size="10485760", enable-http2="true")
   /subsystem=undertow/server=default-server/http-listener=default:write-attribute(name=redirect-socket, value="httpspriv")
   :reload
   ```   
   after this port http (8080) bisa diakses kembali   
 - Configure the HTTPS httpspub listener and set up the public SSL port not requiring the client certificate.
For WildFly 14, instead use enable-http2="false" to avoid error messages in the log.
   ```
   /socket-binding-group=standard-sockets/socket-binding=httpspub:add(port="8442",interface="httpspub")
   /subsystem=undertow/server=default-server/https-listener=httpspub:add(socket-binding="httpspub", security-realm="SSLRealm", max-post-size="10485760", enable-http2="true")
   ```
 - Configure the remoting (HTTP) listener and secure the CLI by removing the http-remoting-connector from using the HTTP port and instead use a separate port 4447.
For WildFly 14, instead use enable-http2="false" to avoid error messages in the log.
   ```
   /subsystem=remoting/http-connector=http-remoting-connector:remove
   /subsystem=remoting/http-connector=http-remoting-connector:add(connector-ref="remoting",security-realm="ApplicationRealm")
   /socket-binding-group=standard-sockets/socket-binding=remoting:add(port="4447")
   /subsystem=undertow/server=default-server/http-listener=remoting:add(socket-binding=remoting, max-post-size="10485760", enable-http2="true")
   ```
 - In order for the web services to work correctly when requiring client certificate, you need to configure the Web Services Description Language (WSDL) web-host rewriting to use the request host.
   ```
   /subsystem=webservices:write-attribute(name=wsdl-host, value=jbossws.undefined.host)
   /subsystem=webservices:write-attribute(name=modify-wsdl-address, value=true)
   #If the server is slow, wait before reloading:
   :reload
   ```
 - To configure the URI encoding, run the following:
   ```
   /system-property=org.apache.catalina.connector.URI_ENCODING:remove()
   /system-property=org.apache.catalina.connector.URI_ENCODING:add(value=UTF-8)
   /system-property=org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING:remove()
   /system-property=org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING:add(value=true)
   :reload
   ```   
   (Failure messages for the two remove commands above are expected if this command is executed for the first time)   

entry hasil update lihat di sini:   
`/opt/wildfly-10.1.0.Final/standalone/configuration/standalone.xml`

## Database

menggunakan docker   
`docker pull mariadb:10.6.4`

example:   
`docker run -p 23306:3306 -itd --name ssmariadb -h ssmariadb -e MARIADB_ROOT_PASSWORD=passw0rd -e MARIADB_DATABASE=ssdb -v C:\my\own\datadir:/var/lib/mysql --net seele_network -d mariadb:10.6.4`

## Konfigurasi Database Driver

```
cp mariadb-java-client-2.7.4.jar $APPSRV_HOME/standalone/deployments/mariadb-java-client.jar
```

## Konfigurasi Data Source
Untuk menambahkan data source agar Signserver bisa menggunakannya, jalankan perintah berikut ini dengan menggunakan Jboss CLI. Data Source yang akan di-deploy ini menggunakan driver dari MariaDB yang dicopy sebelumnya:   
```
jboss-cli.sh
connect

data-source add --name=signserverds --driver-name="mariadb-java-client.jar" --connection-url="jdbc:mysql://ssmariadb:3306/ssdb" --jndi-name="java:/SignServerDS" --use-ccm=true --driver-class="org.mariadb.jdbc.Driver" --user-name="root" --password="passw0rd" --validate-on-match=true --background-validation=false --prepared-statements-cache-size=50 --share-prepared-statements=true --min-pool-size=5 --max-pool-size=150 --pool-prefill=true --transaction-isolation=TRANSACTION_READ_COMMITTED --check-valid-connection-sql="select 1;" --enabled=true
:reload
```

## Instalasi Maven
Maven ini digunakan untuk membangun Signserver dari source. 

`dnf --assumeyes install maven`

## Instalasi Signserver

setelah unduh dan unzip kita build via maven

```
cd /opt/signserver
mvn help:effective-settings

./bin/ant init

# build from source
mvn install -DskipTests
```

## Set Environment Variables

using nano add :
```
nano nano /etc/profile.d/signserver.sh

## signserver.sh 

SIGNSERVER_HOME=/opt/signserver
export SIGNSERVER_HOME
export PATH=$SIGNSERVER_HOME/bin:$PATH
export SIGNSERVER_NODEID=node1
```

## Konfigurasi Deployment (signserver_deploy.properties)

```
cd /opt/signserver
cp conf/signserver_deploy.properties.sample conf/signserver_deploy.properties
nano conf/signserver_deploy.properties

## isi
appserver.home=${env.APPSRV_HOME}
appserver.type=jboss
datasource.jndi-name=SignServerDS

database.name=mysql
database.url=jdbc.mysql://ssmariadb:3306/ssdb
database.username=root
database.password=passw0rd
```

## Deployment

Untuk deployment. Jalankan perintah berikut:

```
cd /opt/signserver
./bin/ant deploy
```

lihat hasil deployment process pada server log di Wildfly, jalankan:

`ls /opt/wildfly/standalone/deployments | grep signserver.ear*`


Verifikasi dengan mengakses Signserver, akses halaman web-nya pada link:

*http://ss.gehirn.org:8080/signserver*

uji melalui CLI dan melihat versi Signserver-nya, Jalankan perintah berikut di CLI:

```
cd /opt/signserver
./bin/signserver getstatus brief all
```

akses halaman administrator dari Signserver, arahkan ke:

http://ss.gehirn.org:8443/signserver/adminweb/

Untuk mengizinkan siapapun mengakses halaman admin tersebut secara sementara, jalankan perintah berikut:

cd /opt/signserver
./bin/signserver wsadmins -allowany

ini mengizinkan administrator mengakses Tab *Workers*, *Global Configuration* dll yang terdapat dalam laman adminweb (http://ss.gehirn.org:8443/signserver/adminweb/)


### commit container

docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]


## referensi 

 - https://blog.goreinnamah.com/blog/2020/06/05/instalasi-signserver-5-2-0/
 - https://en.wikipedia.org/wiki/WildFly
 - https://stackoverflow.com/questions/59466250/docker-system-has-not-been-booted-with-systemd-as-init-system
 - https://doc.primekey.com/signserver520/signserver-installation
 - https://hub.docker.com/_/mariadb?tab=description