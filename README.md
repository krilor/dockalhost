# dockalhost
My local docker dev environment It is simply a docker-compose file with some basic services.

It contains:

* træfic - reverse proxy for my `docker.localhost` hostname
* whoami - just to verify that everything is running
* mariadb - because everybody needs a db
* adminer - to admin the db

## Getting started

### 1 - Create the required networks

```
docker network create -d bridge dev-gateway
docker network create -d bridge dev-backend
```

Check that the networks exists using `docker network ls`

* dev-gateway - the "external" network that links traefik to other public-facing services
* dev-backend - network used between services on the backend

### 2 - Start the services

`docker-compose up -d`

### 3 - Test

Go to [traefik.docker.localhost/] and [whoami.docker.localhost/]

## Adding a new service

### Using docker-compose

```yml
version: '2'
services:
  hello:
    image: emilevauge/whoami
    labels:
      - "traefik.backend=hello"
      - "traefik.frontend.rule=Host:hello.docker.localhost"
      - "traefik.enable=true"
    restart: always
    networks:
      - web

networks:
  web:
    external:
      name: dev-gateway
```

### Using docker run

```
docker run -d --net dev-gateway -l traefik.backend=hello2 -l traefik.frontend.rule=Host:hello2.docker.localhost -l traefik.enable=true emilevauge/whoami
```



## Files

* docker-compose.yml - docker config with the services that is needed
* traefik.toml - traefik config

## Known issues

### Web browser searches instead of going to docker.localhost

To make the web browser recognize the string as a hostname (not a search term), append an `/` at the en of it. This is a known issue.

### Traefik does not start

This can be a lot of things. Check `docker logs traefik`.

#### Chmod?

Chmod for traefik.toml and acme.json must be 600.
```
chmod 600 traefik.toml
```

### Attaching to mariadb does not work

`docker exec -it dockalhost_maria_1 bash`

See [https://stackoverflow.com/questions/35573698/why-does-docker-attach-hang](StackOverflow). This will give you the shell of the mariadb container.
