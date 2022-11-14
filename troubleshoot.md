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