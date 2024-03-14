# prestashop1.7.5.1-docker
image docker for prestashop 1.7.5.1/mariadb

****************************************************************************************************

1: Install Docker Desktop
https://www.docker.com/products/docker-desktop/

****************************************************************************************************

2: Create a file "Dockerfile" in a folder and copy this code inside the file.

FROM prestashop/base:7.2-apache
LABEL maintainer="PrestaShop Core Team <coreteam@prestashop.com>"
ENV PS_VERSION 1.7.5.1

ADD https://www.prestashop.com/download/old/prestashop_1.7.5.1.zip /tmp/prestashop.zip

RUN mkdir -p /tmp/data-ps \
   && unzip -q /tmp/prestashop.zip -d /tmp/data-ps/ \
   && bash /tmp/ps-extractor.sh /tmp/data-ps \
   && rm /tmp/prestashop.zip

RUN apt-get clean && rm -rf /var/lib/apt/lists/*
RUN echo "oui"
RUN service apache2 start

****************************************************************************************************

3: Create a second file "docker-compose.yml" on the same folder and copy this code inside the file.
version: "1"
services:

  # SOURCE : https://hub.docker.com/_/mariadb
  mariadb:
    image: mariadb
    restart: always
    networks:
      - prestashop
    ports:
      - 3306:3306
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_DATABASE=prestashop
      - MYSQL_ROOT_PASSWORD=mycustompassword
  # SOURCE : myself-DOCKERFILE
  prestashop:
    container_name: prestashopkorri
    build:
      context: .
      dockerfile: Dockerfile
    restart: always
    networks:
      - prestashop
    ports:
      - 8080:80
    links:
      - mariadb:mariadb
    depends_on:
      - mariadb
    volumes:
      - ./:/var/www/html:rw
      - ./modules:/var/www/html/modules
      - ./themes:/var/www/html/themes
      - ./override:/var/www/html/override
    environment:
      - PS_DEV_MODE=1
      - PS_INSTALL_AUTO=0
      - DB_NAME=prestashop
      - DB_SERVER=mariadb
      - DB_USER=root
      - DB_PASSWD=mycustompassword

  # SOURCE : https://hub.docker.com/_/phpmyadmin
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    restart: always
    networks:
      - prestashop
    links:
      - mariadb:mariadb
    ports:
      - 8081:80
    depends_on:
      - mariadb
    environment:
      - PMA_HOST=mariadb
      - PMA_USER=root
      - PMA_PASSWORD=mycustompassword

networks:
  prestashop:

volumes:
  db_data:

****************************************************************************************************

4: Run this terminal command :
$ docker compose build

****************************************************************************************************

5: Run this second terminal command :
$ docker compose up

****************************************************************************************************

6: Go for the localhost:8080 on the browser

****************************************************************************************************

7: Start install prestashop

****************************************************************************************************

8: Test the database connection, if it's ok, go for next step.

server : mariadb
name : prestashop
user : root
password : mycustompassword

****************************************************************************************************

9: Finish the installation of prestashop, when it's finished, go on your folder and delete the "install" folder from your global folder, then you can start exploit prestashop.

****************************************************************************************************
