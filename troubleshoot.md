# Troubleshoot

### restart db (if using mariadb then:)

```
systemctl start mariadb
```

### restart wildfly

```
systemctl stop wildfly
systemctl start wildfly
```
or 

```
systemctl restart wildfly
```

### wildfly log location (usually)

```
/opt/wildfly/standalone/log/server.log
```


### Webclient or webadmin signserver cannot be accessed (error code 404 not found)

But wildfly is running..

then login to the server and try run

```bash

cd /opt/signserver/bin
./signserver wsadmins -allowany true
```

try login to the web admin or webclient again

### reload all workers

```reloadallworker```

### found problem with restricted algorithm for certificate

Or if java client apps accessing signserver webservice got error like *"javax.net.ssl.SSLHandshakeException: No supported signature algorithm"*

example:   
```
The input uses a 2048-bit DSA key which is considered a security risk and is disabled
```
edit entry in file `/usr/share/crypto-policies/DEFAULT/java.txt` or `/etc/crypto-policies/back-ends/java.config`

edit entry:   
```
jdk.certpath.disabledAlgorithms=MD2, MD5, DSA, RSA keySize < 2048
```

remove in this case DSA entry if you want to accept 2048-bit DSA key


https://stackoverflow.com/questions/54634060/java-algorithm-constraints-check-failed-on-key-rsa-with-size-of-1024bits