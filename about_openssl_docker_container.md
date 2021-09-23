# Image for openssl lib and tools

menggunakan centos centos8.3.2011

cara run:   

`docker build -t local-openssl:0.0.1 -f Dockerfile.centos_openssl .`

`docker run -it --rm --name openssl_container -v ./usingejbca_cert:/mnt/external local-openssl:0.0.1 bash`