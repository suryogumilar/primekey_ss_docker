# About singserver with maria db container

### example of secrets file for mariadb container 

```bash

## for maria db container
MARIADB_ROOT_PASSWORD=passw0rd
MARIADB_DATABASE=signserverdb
MARIADB_USER=signserver
MARIADB_PASSWORD=passw0rd

## for signserver container
DATABASE_USER=signserver
DATABASE_PASSWORD=passw0rd
```

## Run 

`docker-compose --project-name ss_db -f ./docker-compose_ss_mariadb.yml up --detach`


`docker-compose --project-name ss_db -f ./docker-compose_ss_mariadb.yml up --detach mariadb_ss`

`docker-compose --project-name ss_db -f ./docker-compose_ss_mariadb.yml up --detach signserver-service`

## stop and remove

`docker-compose --project-name ss_db -f ./docker-compose_ss_mariadb.yml down --remove-orphan --volume`

### stop and remove mariadb container

`docker-compose --project-name ss_db -f ./docker-compose_ss_mariadb.yml stop mariadb_ss`

`docker-compose --project-name ss_db -f ./docker-compose_ss_mariadb.yml rm -f mariadb_ss`

## cek lognya


`docker-compose --project-name ss_db -f ./docker-compose_ss_mariadb.yml logs --timestamp --follow mariadb_ss`

`docker-compose --project-name ss_db -f ./docker-compose_ss_mariadb.yml logs --timestamp --follow signserver-service`