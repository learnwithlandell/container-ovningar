# 🐳 WordPress-plattform med 3 containrar

**Kräver att du har [Docker Desktop](https://www.docker.com/products/docker-desktop/) installerat**

Portning av vm-ovningar/ovning-4-vbox-lamp-wp-plattform.md till docker

**(web, db, utils) + färdig wp-config**

---

## 📁 1. Skapa mapp

```
mkdir wp-3container
cd wp-3container
```

---

## 📄 2. Skapa `docker-compose.yml`

```
nano docker-compose.yml
```

Klistra in:

```
services:
  db:
    image: mariadb:10.6
    container_name: wp-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: password
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: wp-web
    depends_on:
      - db
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: password
      WORDPRESS_DB_NAME: wordpress
    ports:
      - "8080:80"
    volumes:
      - wp_data:/var/www/html
      - ./wp-config.php:/var/www/html/wp-config.php

  utils:
    image: debian:stable-slim
    container_name: wp-utils
    restart: always
    command: ["sleep", "infinity"]
    networks:
      - default
    build:
      context: .
      dockerfile: utils.Dockerfile

volumes:
  db_data:
  wp_data:
```

---

## 🧰 3. Skapa utils-container (verktyg)

```
nano utils.Dockerfile
```

Innehåll:

```
FROM debian:stable-slim

RUN apt update && apt install -y \
    curl \
    iputils-ping \
    mariadb-client \
    nano

CMD ["sleep", "infinity"]
```

Utils-containern kan du använda för att testa:

```
docker exec -it wp-utils bash
ping wp-web
mysql -h wp-db -u wpuser -ppassword wordpress
```

---

## 📝 4. Skapa färdig `wp-config.php`

```
nano wp-config.php
```

Innehåll:

```
<?php

define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'password' );
define( 'DB_HOST', 'db' );

define( 'DB_CHARSET', 'utf8' );
define( 'DB_COLLATE', '' );

define( 'WP_HOME', 'http://localhost:8080' );
define( 'WP_SITEURL', 'http://localhost:8080' );

/* Unika nycklar */
define('AUTH_KEY',         'put your unique phrase here');
define('SECURE_AUTH_KEY',  'put your unique phrase here');
define('LOGGED_IN_KEY',    'put your unique phrase here');
define('NONCE_KEY',        'put your unique phrase here');
define('AUTH_SALT',        'put your unique phrase here');
define('SECURE_AUTH_SALT', 'put your unique phrase here');
define('LOGGED_IN_SALT',   'put your unique phrase here');
define('NONCE_SALT',       'put your unique phrase here');

$table_prefix = 'wp_';

define( 'WP_DEBUG', false );

if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

require_once ABSPATH . 'wp-settings.php';
```

(WordPress kommer hoppa över installationsguiden eftersom configen redan finns.)

---

## ▶️ 5. Starta allt

```
docker compose up -d
```

Kolla status:

```
docker compose ps
```

---

## 🌐 6. Öppna WordPress

Öppna:

**[http://localhost:8080](http://localhost:8080)**

WordPress är nu färdigconfigurerat och redo.
