---
layout: wiki
title: PHP WordPress Remote Debugging
cate1: WordPress
cate2: Debugging
description: PHP WordPress Remote Debugging
keywords: Debugging
---

# PHP WordPress Remote Debugging

## Setup Instructions

### Prerequisites
1. **Download PHPStorm**
2. **Download Docker Desktop**

### Docker Setup
1. **Use Bitnami/WordPress Container**
    ```bash
    docker-compose up -d [docker-compose.yml]
    ```
2. **Access WordPress Admin**
    ```plaintext
    user: bitnami
    ```

### Install Vim
```bash
docker ps
docker exec -u 0 -it [container_id] /bin/bash
apt-get update && apt-get install autoconf make gcc
pecl install xdebug
```

### Configure XDebug
Edit `php.ini`
```plaintext
cd /opt/bitnami/php/etc/php.ini

[XDebug]
zend_extension = xdebug
xdebug.mode = debug
xdebug.client_host=host.docker.internal
xdebug.start_with_request=yes
```

Copy and restart the container
```bash
Copy code
docker cp [container_id]:/opt/bitnami/ .
docker restart [container_id]
```

### WordPress Plugins and Downgrade
  + Manually install all necessary plugins.
  + Downgrade WordPress if needed in docker-compose.yml.

### Extract /opt Directory
  + Extract /opt directory to the local instance.

### Debugging
  + Run and set a breakpoint in PHPStorm.

# Logging

## Update MariaDB Logging
```bash
docker exec -u 0 -it [mariadb_container_id] /bin/bash
nano /opt/bitnami/mariadb/conf/my.cnf
```

## Modify log location
```plaintext
replace log_error with /tmp/mysqld.log
```

## Restart MariaDB
```bash
docker restart [mariadb_container_id]
```

# WordPress Setup
`docker-compose.yml`
```yaml
version: '2'
services:
  mariadb:
    image: docker.io/bitnami/mariadb:10.3
    volumes:
      - 'mariadb_data:/bitnami/mariadb'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_wordpress
      - MARIADB_DATABASE=bitnami_wordpress
  wordpress:
    image: docker.io/bitnami/wordpress:6
    ports:
      - '9200:9200'
      - '80:8080'
      - '443:8443'
    volumes:
      - 'wordpress_data:/bitnami/wordpress'
    depends_on:
      - mariadb
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - WORDPRESS_DATABASE_HOST=mariadb
      - WORDPRESS_DATABASE_PORT_NUMBER=3306
      - WORDPRESS_DATABASE_USER=bn_wordpress
      - WORDPRESS_DATABASE_NAME=bitnami_wordpress
    extra_hosts:
      - "host.docker.internal:host-gateway"
volumes:
  mariadb_data:
    driver: local
  wordpress_data:
    driver: local
```