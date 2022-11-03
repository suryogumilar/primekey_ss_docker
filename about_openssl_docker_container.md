# Image for openssl lib and tools

menggunakan centos centos8.3.2011

cara run:   

`docker build -t local-openssl:0.0.1 -f Dockerfile.centos_openssl .`

`docker run -it --rm --name openssl_container -v ./usingejbca_cert:/mnt/external local-openssl:0.0.1 bash`


### troubleshoot 

jika menghadapi error `Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: No URLs in mirrorlist` saat build maka tambahkan baris: 

```
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
```