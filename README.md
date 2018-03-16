# Lando Image for OroCommerce
Extends Lando's base PHP image with OroCommerce PHP extension dependencies.

I hope to create a Lando Recipe to automate this setup. For now, here is how you can get OroCommerce running on Lando:

1. Create or browse to your OroCommerce project root.
2. Modify or create your `.lando.yml`
    - Override the PHP image with the OroCommerce image - example provided blow
    - Implement a Symfony-compatible NGINX - example provided below
    - Increase PHP's memory limit - example provided below

```yml
# .lando.yml Example
name: oro-test

proxy:
  nginx:
    - oro.lndo.site

services:
  appserver:
    type: php:7.1
    via: nginx
    ssl: true
    webroot: web
    xdebug: true
    ####
    # This Override is where we work our magic
    overrides:
      services:
        image: improper/lando-orocommerce-php:7.1-fpm
    config:
      server: ${LANDO_MOUNT}/lando/nginx/default.conf
      conf: ${LANDO_MOUNT}/lando/php/php.ini
    # End of OroCommerce override
    ###
```

```ini
; php.ini
memory_limit=2048M
```

```
#######
# LANDO NGINX for Symfony
# For future reference, we will want to automate the placement of this file using:
#       /bin/bash -c "envsubst < $LANDO_MOUNT/dev/lando/config/nginx/default.template > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"

server {
    ########
    ## LANDO
    # For inspiration, SSH into the nginx service via `lando ssh monin-americas nginx` and take a look at /etc/nginx/conf.d/default.template

    listen       443 ssl;
    listen       80;
    listen   [::]:80 default ipv6only=on;
    server_name  localhost;

    ssl_certificate      /certs/cert.pem;
    ssl_certificate_key  /certs/cert.key;

    root   ${LANDO_WEBROOT};
    index index.php index.html index.htm;

 location / {
        # try to serve file directly, fallback to app.php
        try_files $uri /app.php$is_args$args;
    }

    location ~ ^/(app|app_dev|config|install)\.php(/|$) {

        fastcgi_pass fpm:9000;
        fastcgi_index  index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_connect_timeout 300s;
        fastcgi_send_timeout 300s;
        fastcgi_read_timeout 300s;
        include fastcgi_params;

        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```
