# Instalasi signserver

instalasi manual. menggunakan image centos8.3.2011

## container

`docker run -itd --privileged -v C:\insta:/mnt/insta -p 28080:8080 -p 28443:8443 --name insta_ss centos:centos8.3.2011`

`docker exec -it insta_ss bash`

catatan: docker container sebagai testing instalasi harus run in privileged mode karena butuh systemctl jika tidak akan dapat error:
> System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down


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
dnf install epel-release
dnf update
```

instal script service jika belum ada

`dnf install initscripts`

install nano untuk edit script

`dnf install nano`

install unzip kalau belum ada

`dnf install unzip`

instal wget

`dnf install wget`

## instalasi

### instal open JDK 8

`dnf install java-1.8.0-openjdk-devel`

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
WILDFLY_VERSION=25.0.0.Final
WILDFLY_FILENAME=wildfly-$WILDFLY_VERSION
WILDFLY_ARCHIVE_NAME=$WILDFLY_FILENAME.tar.gz
WILDFLY_DOWNLOAD_ADDRESS=https://github.com/wildfly/wildfly/releases/download/$WILDFLY_VERSION/$WILDFLY_ARCHIVE_NAME
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
```

sebagai user **wildfly** start melaui command : 

`/opt/wildfly/bin/launch.sh $WILDFLY_MODE $WILDFLY_CONFIG $WILDFLY_BIND`

##### tes if wildfly is running

open browser buka http://localhost:8080 

## referensi 

 - https://blog.goreinnamah.com/blog/2020/06/05/instalasi-signserver-5-2-0/
 - https://en.wikipedia.org/wiki/WildFly
 - https://stackoverflow.com/questions/59466250/docker-system-has-not-been-booted-with-systemd-as-init-system