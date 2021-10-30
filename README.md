# project_docker_blackfire

# creation project blackfilre, docker-compose, clone demo_symfony

docker exec www_docker_symfony composer create-project symfony/symfony-demo project

docker-compose up -d

# Mise à jour du droits ch mode avec

sudo chmod -R 777 project/var/log/

# Activer la sonde PHP de Bblackfire dans DockerFile

FROM php:7.4-fpm

RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
    && architecture=$(uname -m) \
 && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/$architecture/$version \
 && mkdir -p /tmp/blackfire \
 && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp/blackfire \
 && mv /tmp/blackfire/blackfire-\*.so $(php -r "echo ini_get ('extension_dir');")/blackfire.so \
 && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8307\n" > $PHP_INI_DIR/conf.d/blackfire.ini \
 && rm -rf /tmp/blackfire /tmp/blackfire-probe.tar.gz

# faire builder docker, pour ajouté la sonde blackfire

docker-compose build

# stop container docker

docker-compose stop

# start docker

docker-compose up -d --build

# ajoute les Informations d'identification personnelles de Blackfire dans un fichier .env

# déclarer l'agent Blackfire en tant que service

blackfire:
image: blackfire/blackfire:2
container*name: blackfire_docker_symfony
ports: ["8307"]
environment: # Exposes BLACKFIRE*\* environment variables from the host
BLACKFIRE_SERVER_ID: $BLACKFIRE_SERVER_ID
BLACKFIRE_SERVER_TOKEN: $BLACKFIRE_SERVER_TOKEN
BLACKFIRE_CLIENT_ID: $BLACKFIRE_CLIENT_ID
BLACKFIRE_CLIENT_TOKEN: $BLACKFIRE_CLIENT_TOKEN

# Validez le ficher.env

docker-compose --env-file .env up -d
