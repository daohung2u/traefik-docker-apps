Triển khai traefik docker services

Docker services
```
* Traefik
* Mysql
* Whoami
* Redmine
* Wordpress
```

Sample
```yaml
version: "3"

services:
  traefik:
    build: ./bin/traefik
    container_name: traefik
    restart: always
    ports:
    - 80:80
    - 443:443
    - 8080:8080
    command:
      # traefik managed entry point like port 80
      - --entrypoints.web.address=:80

      # traefik managed entry poiint like port 443
      - --entrypoints.websecure.address=:443

      # traefik managed docker
      - --providers.docker

      # traefik daskboard like localhost:8080, use --api, dashboard with basic_auth or api.ensure
      - --api.insecure

      # traefik managed tls for verify let encrypt certificate
      - --certificatesresolvers.mytlschallenge.acme.tlschallenge=true

      # email when use let encrypt
      - "--certificatesresolvers.myhttpchallenge.acme.email=daohung.tg@gmail.com"

      # store let's encrypt cert
      - "--certificatesresolvers.mytlschallenge.acme.storage=/acme.json"
    volumes:
      # storage outside volumn
      - ./acme.json:/acme.json

      # treafik access docker
      - /var/run/docker.sock:/var/run/docker.sock:ro

    labels:
      # Dashboard
      - "traefik.http.routers.traefik.service=api@internal"

  app:
    image: containous/whoami
    labels:
      # traefik read host name and route to this if treafik receive request like Host
      - "traefik.http.routers.app.rule=Host(`web-app.local`)"

  php-fpm:
    build: ./bin/php-fpm
    container_name: php-fpm
    restart: always

  mysql-redmine:
    build: ./bin/mysql
    restart: always
    container_name: mysql-redmine
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: redmine
    volumes:
       - ./data/mysql-redmine:/var/lib/mysql

  mysql-wordpress:
    build: ./bin/mysql
    restart: always
    container_name: mysql-wordpress
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: wordpress
    volumes:
    - ./data/mysql-wordress:/var/lib/mysql

  redmine:
    build: ./bin/redmine
    container_name: redmine
    restart: always
    links:
      - mysql-redmine
    environment:
      REDMINE_DB_MYSQL: mysql-redmine
      REDMINE_DB_PASSWORD: 123456
      REDMINE_SECRET_KEY_BASE: supersecretkey
    volumes:
    - ./www/redmine:/usr/src/redmine/files
    labels:
      # traefik read host name and route to this if treafik receive request like Host
      - "traefik.http.routers.redmine.rule=Host(`redmine.local`)"


  wordpress:
    image: wordpress
    container_name: wordpress
    restart: always
    links:
      - mysql-wordpress
    environment:
      WORDPRESS_DB_HOST: mysql-wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: 123456
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - ./www/wordpress:/var/www/html
    labels:
      # traefik read host name and route to this if treafik receive request like Host
      - "traefik.http.routers.wordpress.rule=Host(`wordpress.local`)"

```
