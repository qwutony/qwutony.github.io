---
layout: wiki
title: Remote Debugging - VSCode
cate1: WordPress
cate2: Debugging
description: PHP WordPress Remote Debugging
keywords: Debugging
---

# PHP WordPress Remote Debugging

## Requirements
```
VSCode
PHP XDebug Extension installed in VSCode
Docker
Bitnami/Wordpress container (Docker File)
```

## 1. Setup WordPress Docker instance
```
Copy docker-compose.yml file to Kali instance
sudo docker-compose up -d

sudo docker ps
sudo docker exec -u 0 -it [id] /bin/bash
apt-get update && apt-get install autoconf make gcc
pecl install xdebug
sudo apt-get install vim
cd /opt/bitnami/php/etc
vim php.ini
```

## 2. Update XDebug File
```
php.ini
  zend_extension = xdebug
  xdebug.mode = debug
  xdebug.client_host = host.docker.internal
  xdebug.client_port = 9003
  xdebug.output_dir = /tmp
  xdebug.remote_handler = dbgp
  xdebug.start_with_request = yes
```
## 3. Copy source code from Docker instance (or wherever)
```
docker cp 67a1e7f51932:/opt/bitnami/ .

Alternatively, we can use rsync to constantly update the plugins from docker instance:
alias drsync="rsync -e 'docker exec -i'"       [as root on local machine]
drsync -av c6bf7b9c724b:/opt/bitnami /tmp/test [as root on local machine]
```

Note; the /opt/bitnami wp-content file is just a symlink to /bitnami/wp-contents, remove the symlink and pull /bitnami/wp-contents to bitnami folder for plugin debugging.

## 4. VSCode Setup
Ensure PHP XDebug Extension is installed
Create launch.json file (should be in "Run and Debug", there is either an option to create one (select PHP), or press the cog icon to open launch.json)
Modify the launch.json file as below or use the one at the very bottom of this markdown file:
```
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "hostname": "0.0.0.0",
            "pathMappings": {
                "/bitnami/wordpress/wp-content": "${workspaceFolder}/bitnami/wordpress/wp-content",
                "/opt/bitnami/": "${workspaceFolder}/bitnami"
            },
            "log": true
        },
```
Modify the following values to suit individual taste:
 + hostname: Needs to be "0.0.0.0" since Docker instances listen via IPv4
 + port: Needs to be the same as XDebug in php.ini file
 + pathMappings: LEFT (Docker directory to /wp-contents), RIGHT (Local directory to /wp-contents)

## 6. Debugging Complete

```
docker restart [id]
```

## Logging
```
docker exec -u 0 -it [mariadb] /bin/bash
nano /opt/bitnami/mariadb/conf/my.cnf
replace log_error with something like /tmp/mysqld.log
docker restart [mariadb]
```
===================================================================

# Files

## docker-compose.yml
```
version: '2'
services:
  mariadb:
    image: docker.io/bitnami/mariadb:10.3
    volumes:
      - 'mariadb_data:/bitnami/mariadb'
    environment:
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
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
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
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

Note: For windows it isnt able to resolve the docker.host.internal like it is for OSX, to fix this, modify the extra_hosts - host.docker.internal to be 
host.docker.internal:local-ip, verify this by using curl or wget host.docker.internal. If its incorrect it will error about a CA authority

## launch.json
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "hostname": "0.0.0.0",
            "pathMappings": {
                "/bitnami/wordpress/wp-content": "${workspaceFolder}/bitnami/wordpress/wp-content",
                "/opt/bitnami/": "${workspaceFolder}/bitnami"
            },
            "log": true
        },
        {
            "name": "Launch currently open script",
            "type": "php",
            "request": "launch",
            "program": "${file}",
            "cwd": "${fileDirname}",
            "port": 0,
            "runtimeArgs": [
                "-dxdebug.start_with_request=yes"
            ],
            "env": {
                "XDEBUG_MODE": "debug,develop",
                "XDEBUG_CONFIG": "client_port=${port}"
            }
        },
        {
            "name": "Launch Built-in web server",
            "type": "php",
            "request": "launch",
            "runtimeArgs": [
                "-dxdebug.mode=debug",
                "-dxdebug.start_with_request=yes",
                "-S",
                "localhost:0"
            ],
            "program": "",
            "cwd": "${workspaceRoot}",
            "port": 9003,
            "serverReadyAction": {
                "pattern": "Development Server \\(http://localhost:([0-9]+)\\) started",
                "uriFormat": "http://localhost:%s",
                "action": "openExternally"
            }
        }
    ]
}
```
