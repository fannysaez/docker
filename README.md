# Guide Docker Complet ![Docker](https://img.shields.io/badge/docker-ready-blue?logo=docker) ![Compose](https://img.shields.io/badge/compose-supported-blueviolet?logo=docker)

## Sommaire

- [Pré-requis](#pré-requis)
- [Commandes Docker de base](#commandes-docker-de-base)
- [Volumes & Réseaux](#volumes--réseaux)
- [Docker Compose](#docker-compose)
- [Contexte du projet Node.js/MariaDB/PhpMyAdmin](#contexte-du-projet-nodejsmariadbphpmyadmin)
- [Livrables attendus](#livrables-attendus)
- [Commandes utiles](#commandes-utiles)

---

## Pré-requis

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installé (Windows/Mac/Linux)
- [Docker Compose](https://docs.docker.com/compose/) (inclus dans Docker Desktop)
- Droits administrateur sur votre machine

Vérifiez l’installation :

```shell
docker --version
docker compose version
```

---

## Commandes Docker de base

### Pull

Récupère une image depuis Docker Hub :

```shell
docker pull <image>
docker pull <image:tag>
```

### Voir les containers/process

```shell
docker ps
docker ps -a   # voir aussi les containers arrêtés
```

### Run

Lancer un container :

```shell
docker run <image>
docker run -it <image>           # mode interactif
docker run <image> sleep 5       # exécuter une commande
docker run -d <image>            # mode détaché (en fond)
docker attach <container>        # rattacher un container détaché
```

### Ports

Mapper un port du container sur le localhost :

```shell
docker run -p 80:5000 <image>
```

### Variables d'environnement

```shell
docker run -e APP_COLOR=teal <image>
```

### Entrypoint et CMD

Override possible :

```shell
docker run --entrypoint <newentrypoint> <image>
```

### Exec

Exécuter une commande dans un container existant :

```shell
docker exec <container> <cmd>
```

### Stop

Arrêter un container :

```shell
docker stop <container>
```

### Images

Lister les images installées :

```shell
docker images
```

### Remove

Supprimer un container ou une image :

```shell
docker rm <container>
docker rmi <image>
```

### Inspect

Analyser un container :

```shell
docker inspect <container>
```

### Logs

Voir les logs d’un container :

```shell
docker logs <container>
```

### Build & Push

Construire et publier une image :

```shell
docker build -t maksim/monimage .
docker push maksim/monimage
```

### History

Voir les layers d’une image :

```shell
docker history <image>
```

---

## Volumes & Réseaux

### Volumes

Créer un volume :

```shell
docker volume create <volumename>
```

Persistance des données :

```shell
docker run -v /path/to/savelocal:/path/in/container <image>
docker run -v <volumename>:/var/lib/mysql <image>
```

Version détaillée :

```shell
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql <image>
```

### Réseaux

Lister les réseaux :

```shell
docker network ls
```

Créer un réseau :

```shell
docker network create --driver bridge --subnet 182.18.0.0/16 <networkname>
```

---

## Docker Compose

Définir une stack multi-container dans `docker-compose.yml` ou `compose.yml`.

Exemple :

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

Commandes :

```shell
docker compose build
docker compose up
docker compose down
docker compose up --build
docker compose -f ./docker/compose.yml up
docker compose exec mon_service bash
```

---

## Contexte du projet Node.js/MariaDB/PhpMyAdmin

Vous disposez du code source d’une application Node.js.

**Objectif :**  
Créer une image Docker qui :

- Contient le code source de l’application
- Installe les dépendances (`npm install`)
- Démarre automatiquement le serveur ExpressJS

**Étapes :**

1. **Créer un `Dockerfile`** pour builder l’application Node.js.
2. **Créer un `compose.yml`** pour lancer :
   - L’application Node.js
   - Une base de données MariaDB
   - PhpMyAdmin pour visualiser la base

### Exemple de Dockerfile

```dockerfile
# filepath: ./Dockerfile
FROM node:20-alpine

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

### Exemple de compose.yml

```yaml
# filepath: ./compose.yml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: node_app
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
      - DB_USER=demo
      - DB_PASSWORD=demo
      - DB_NAME=demo
    depends_on:
      - db

  db:
    image: mariadb:11
    container_name: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: demo
      MYSQL_USER: demo
      MYSQL_PASSWORD: demo
    ports:
      - "3306:3306"
    volumes:
      - dbdata:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: pma
    environment:
      PMA_HOST: db
      PMA_USER: demo
      PMA_PASSWORD: demo
    ports:
      - "8080:80"
    depends_on:
      - db

volumes:
  dbdata:
```

---

## Livrables attendus

- `Dockerfile` (pour l’application Node.js)
- `compose.yml` (pour la stack complète)
- Documentation (ce fichier README.md)

---

## Commandes utiles

Pour builder et lancer la stack :

```shell
docker compose build
docker compose up
```

Pour arrêter la stack :

```shell
docker compose down
```

Pour accéder à PhpMyAdmin :  
[http://localhost:8080](http://localhost:8080)  
Login : `demo` / Mot de passe : `demo`  
Serveur : `db`

---

**Astuce :**  
Pour relancer la stack après modification du code :

```shell
docker compose up --build
```

---

<!-- > **Modalités pédagogiques :**  
> Travail individuel, sur 2 jours.  
> Les fichiers Dockerfile et compose.yml doivent être rendus et accompagnés d’une documentation simple.

> **Modalités d’évaluation :**  
> Le Dockerfile et compose.yml sont fonctionnels et répondent à l’objectif décrit. -->
