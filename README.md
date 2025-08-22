# Pull

Récupère une image depuis docker hub

```shell
    docker pull <image>
    docker pull <image:tag>
```

## PS

Voir la liste des containers/process docker

```shell
    docker ps
    docker ps -a (pour voir les containers à l'arret aussi)
```

## Run

Lance un container depuis une image

```shell
    docker run <container>
    # Mode interactif
    docker run -it <container>
```

Append a command

```shell
    docker run <container> sleep 5
```

"detach" option pour laisser le container tourner en fond

```shell
    docker run -d <container>
```

Pour le "rattacher

```shell
    docker attach <container>
```

## Port

Map le port du container avec le localhost

```shell
    docker run -p 80:5000 <container>
```

## Env

Lancer un container en modifiant le .env

```shell
    docker run -e APP_COLOR=teal <container>
```

## Entrypoint et Cmd

Si un ENTRYPOINT et un CMD sont préciser dans le Dockerfile on peut les overide

```shell
    docker run --entrypoint <newentrypoint> <container>
```

## Exec

Lance un container et execute une commande

```shell
    docker exec <container> <cmd>
```

## Stop

Stop un container qui tourne

```shell
    docker stop <container>
```

## Images

Affiche la liste des images installées

```shell
    docker images
```

## Remove

Supprimer un container ou une image

```shell
    docker rm <container>
    docker rmi <image>
```

## Inspect

Analyser un container et ses variables d'environnement

```shell
    docker inspect <container>
```

## Log

Voir les logs d'un container qui tourne en tache de fond

```shell
    docker logs <container>
```

## Build

Build sa propre image puis la push sur docker hub

```shell
    docker build Dockerfile -t maksim/monimage
    docker push maksim/monimage
```

## History

Permet de voir les layers et leur poids

```shell
    docker history <image>
```

## Networks

```shell
    docker run <image>                      (Bridge: chaque container à sa propre IP)
    docker run <image> --network=none       (aucun: le container est isolé)
    docker run <image> --network=host       (Host: tout est mappé sur le local)
```

Lister les networks

```shell
    docker network ls
```

Créer un network

```shell
    docker network create --driver bridge --subnet 182.18.0.0/16 <networkname>
```

## File System

/var/lib/docker
/aufs
/containers
/image
/volumes

## Volume

Créer un volume

```shell
    docker volume create <volumename>
```

Gérer la persistance des données même après suppression du container

```shell
# "bind mount parcequ'on bind un dossier local"
    docker run -v /path/to/savelocal:/path/of/containerdata <container>
    ou
# "volume mount parcequ'on bind un volume docker"
    docker run -v <volumename>:/var/lib/mysql
```

Verbose version

```shell
    docker run \
    --mount type=bind,source=/data/mysql,target=/var/lib/mysql <image>
```

```shell
docker container prune
docker image prune # supprime les "danglings"
# docker image prune -a #supprime toutes les images non-utilisé actuellement

# Supprime tout ce qui n'est pas utilisé
docker system prune -a

docker inspect [container_name]
```

Petite astuce assez connue pour que votre user dans le container soit le même que sur votre machine

```shell
docker run -it -u $(id -u):$(id -g) -v "$PWD:/app" ubuntu bash
```

Créer un volume avec le dossier courant

```shell
# Linux ou WSL
-v $(pwd):/app
# Powershell
-v ${PWD}:/app
```

## Docker compose

Système permettant de définir dans un docker-compose.yml (ou "compose.yml")
un ensemble de "services" qui doivent être lancés ensemble.
Très pratique pour avoir une "stack" de containers réutilisable (app + db + phpmyadmin...).

Voici un exemple qui lancera 2 containers:

- un container python qui execute une commande
- un container PostgreSQL

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: demo_app
    working_dir: /usr/src/app
    depends_on:
      - db

  db:
    image: mariadb:11
    container_name: demo_db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: demo
      MYSQL_USER: demo
      MYSQL_PASSWORD: demo
    ports:
      - "3306:3306"
    volumes:
      - monvolume:/data

volumes:
  monvolume:
```

On assume toujours que le dossier courant contient un fichier docker-compose.yml ou compose.yml

```shell
docker compose build
# Lancer tous les services de la stack
docker compose up
# Couper les services de la stack
docker compose down
# "Monte" la stack en forçant un rebuild, utile si une image a changée
docker compose up --build
```

Si le fichier "compose.yml" se trouve dans un autre dossier, on peut le préciser:

```shell
docker compose -f ./docker/compose.yml up
```

```shell
# rentrer dans un contenaire service de la stack
docker compose exec mon_service bash
```

Si un fichier s'appelle docker-compose.override.yaml
Permet d'override certaines lignes du docker-compose.

---