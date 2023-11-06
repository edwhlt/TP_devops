# TP1 - Docker

## 1-1 Document your database container essentials: commands and Dockerfile:

- on utilise l'image postgres:14.1-alpine comme image de base (Alpine est une version légère de Linux)
- On définis des variables d'environnement pour le nom de la base de données, l'utilisateur et le mot de passe.
- On copie deux scripts SQL dans un répertoire spécial (/docker-entrypoint-initdb.d/). Tout les scripts dans ce répertoire seront exécuté par PostgreSQL lors du démarrage du conteneur

## 1-2 Why do we need a multistage build? And explain each step of this dockerfile:

Le build multi-étapes en Docker sert principalement à deux choses :
- Optimiser la taille de l'image finale : La première étape peut inclure tous les outils et fichiers nécessaires pour construire l'application (comme un JDK pour Java ou un compilateur pour C++), qui peuvent être volumineux. La seconde étape ne conserve que ce qui est nécessaire pour exécuter l'application (comme un JRE pour Java), réduisant ainsi la taille de l'image finale.
- Séparer les étapes de construction et d'exécution : Cela permet d'avoir une séparation claire entre la compilation de l'application et son exécution, rendant le Dockerfile plus propre et organisé.

Étape de construction (build) pour l'API backend:
`FROM maven:3.8.6-amazoncorretto-17 AS myapp-build`
Cette ligne indique qu'on utilise l'image `maven:3.8.6-amazoncorretto-17` comme base pour cette étape de construction. C'est une image contenant Maven pour la gestion de projets Java, ainsi que l'Amazon Corretto 17 JDK (une distribution de OpenJDK).
AS myapp-build donne un nom à cette étape, qui sera utilisé dans la suite du Dockerfile.

`ENV MYAPP_HOME /opt/myapp`
Cette ligne définit une variable d'environnement `MYAPP_HOME` avec la valeur `/opt/myapp`.

`WORKDIR $MYAPP_HOME`
Cette commande change le répertoire de travail dans l'image à `/opt/myapp` (la valeur de `MYAPP_HOME`).

`COPY pom.xml .`
Cette ligne copie le fichier `pom.xml` vers le répertoire de travail dans l'image (c'est-à-dire `/opt/myapp`).

`COPY src ./src`
Cette commande copie le répertoire `src` (qui contient le code source de votre application) du contexte de construction vers le répertoire `/src` dans l'image

## 1.3 Document docker-compose most important commands:
`docker-compose up` : Construire et démarrer les services.
`docker-compose down` : Arrête et supprime les services.
`docker-compose build` : Construit ou reconstruit les services.
`docker-compose logs` : Affiche la sortie des conteneurs.
`docker-compose ps` : Liste les conteneurs.

## 1-4 Document your docker-compose file:
Pour `build` :, les chemins (`./backend`, `./httpd`) doivent pointer vers les répertoires contenant les fichiers Docker respectifs.
La directive `networks` : est utilisée pour connecter les services dans un réseau personnalisé pour la communication entre les conteneurs.

## 1-5 Document your publication commands and published images in dockerhub:
Connexion a Docker Hub avec `docker login`
Ensuite on doit préciser un tag pour pouvoir push ensuite `docker tag my-database <USERNAME>/my-database:1.0`
Enfin on push avec la commande `docker push <USERNAME>/my-database:1.0`

## Tips questions

### Why should we run the container with a flag -e to give the environment variables?

Utiliser le drapeau -e permet de définir ces variables lors de l'exécution du conteneur.
Cela permet de personnaliser le comportement de l'application sans avoir à modifier son image.

### Why do we need a volume to be attached to our postgres container?

Il est important de conserver les données de la base de données de manière persistante, même si le conteneur est arrêté ou supprimé.
En attachant un volume au conteneur PostgreSQL, on s'assure que les données de la base de données sont conservées et peuvent être sauvegardées ou restaurées.

### Why do we need a reverse proxy?

Le reverse proxy permet de centraliser la gestion des demandes clients (par exemple sur un serveur httpd) et de les rediriger vers les serveurs appropriés (sur le port 8080) en fonction de règles de routage

### Why is docker-compose so important?

Le docker compose simplifie le déploiement d'applications contenant plusieurs services Docker,
en automatisant la création, la configuration et la liaison des conteneurs. C'est un outil essentiel pour la gestion d'applications multi-conteneurs.

### Why do we put our images into an online repo?

Placer des images de conteneur dans Docker Hub,
permet de les partager, de les distribuer et de les rendre accessibles à
d'autres utilisateurs ou membres de l'équipe directement en ligne.

# TP2 - Github Actions

## 2-1 What are testcontainers?

Testcontainers est une bibliothèque Java qui permet d'exécuter des conteneurs Docker pendant les tests. Dans cet exemple, nous utilisons le conteneur PostgreSQL pour attacher notre application lors des tests. Lorsque vous exécutez la commande "mvn clean verify", un conteneur Docker est lancé pendant l'exécution des tests, ce qui est très pratique pour effectuer des tests d'intégration.

## 2-2 Document your Github Actions configurations.

```
name: CI devops 2023

on:
  push:
    branches:
      - develop
      - master

jobs:
  test-backend:
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0
        # Cette étape récupère le code depuis le dépôt GitHub

      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
          java-version: 17
          distribution: 'adopt'
        # Cette étape configure Java JDK 17 pour l'environnement de build

      - name: Build and test with Maven
        run: mvn clean verify --file backend/pom.xml
        # Cette étape exécute la construction et les tests en utilisant Maven
```

## 2-3 Document your quality gate configuration.

On change la dernière étape du jobs test-backend
```
run: mvn -B verify sonar:sonar -Dsonar.projectKey=edwhlt_TP_devops -Dsonar.organization=edwhlt -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file backend/pom.xml
# Cette étape exécute la construction et les tests en utilisant Maven, puis envoie les résultats à SonarCloud pour l'analyse statique du code
# on précise quel projet sonar, quelle organisation ainsi que le token stocké dans les secrets
```

# TP 3 - Ansible

## 3-1 Document your inventory and base commands

```
all:  # Groupe "all" contenant des variables globales et des groupes d'hôtes.
 vars:
   ansible_user: centos  # Utilisateur SSH pour se connecter aux hôtes du groupe "all".
   ansible_ssh_private_key_file: ~/.ssh/id_rsa  # Clé privée SSH utilisée pour les connexions aux hôtes.
 children:
   prod:  # Groupe d'hôtes "prod".
     hosts: edwin.helet.takima.cloud  # Liste des hôtes dans le groupe "prod".
```

## 3-2 Document your playbook

```
- name: Launch Docker containers  # Nom du playbook, qui décrit son objectif.
  hosts: all  # Groupe d'hôtes cible pour l'exécution de ce playbook (tous les hôtes).
  gather_facts: false  # Désactive la collecte de faits (facts) pour accélérer l'exécution.

  become: true  # Active l'utilisation de privilèges d'administration (sudo) pour les tâches.

  roles:
    - install-docker  # Inclut le rôle "install-docker" pour installer Docker sur les hôtes.
    - create-docker-network  # Inclut le rôle "create-docker-network" pour créer un réseau Docker.
    - launch-database  # Inclut le rôle "launch-database" pour lancer la base de données.
    - launch-app  # Inclut le rôle "launch-app" pour lancer l'application.
    - launch-frontend  # Inclut le rôle "launch-frontend" pour lancer le frontend de l'application.
    - launch-proxy  # Inclut le rôle "launch-proxy" pour lancer un proxy (éventuellement inversé).
```

Ensuite toutes les tasks sont définits dans les différents roles, par exemple la task pour `install-docker`:

```
- name: Install device-mapper-persistent-data
  # Installe le package 'device-mapper-persistent-data' en utilisant le module 'yum'.
  yum:
    name: device-mapper-persistent-data
    state: latest
  # Assure que le package est mis à jour vers la dernière version disponible (state: latest).

- name: Install lvm2
  # Installe le package 'lvm2' en utilisant le module 'yum'.
  yum:
    name: lvm2
    state: latest
  # Assure que le package 'lvm2' est mis à jour vers la dernière version disponible (state: latest).

- name: Add Docker repository
  # Ajoute le référentiel Docker en utilisant la commande 'yum-config-manager'.
  command:
    cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
  # Configure le système pour télécharger les paquets Docker depuis le référentiel officiel de Docker pour CentOS.

- name: Install Docker
  # Installe le package 'docker-ce' (Docker Community Edition) en utilisant le module 'yum'.
  yum:
    name: docker-ce
    state: present
  # Assure que le package 'docker-ce' est installé.

- name: Make sure Docker is running
  # Vérifie que le service Docker est en cours d'exécution en utilisant le module 'service'.
  service: name=docker state=started
  # Démarre le service Docker s'il n'est pas déjà en cours d'exécution.
  tags: docker
  # Associe la balise (tag) 'docker' à cette tâche pour permettre son exécution sélective en fonction des balises utilisées lors de l'exécution du playbook.
```

## 3-3 Document your docker_container tasks configuration

Exemple de docker_container tasks, le backend:

```
- name: Launch app container
  # Nom de la tâche : "Lancer le conteneur de l'application".
  docker_container:
    # Utilise le module 'docker_container' pour gérer les conteneurs Docker.
    name: backend-devops
    # Nom du conteneur : "backend-devops".
    image: edwhlt/tp_devops-backend:latest
    # Image Docker à utiliser pour le conteneur, en utilisant la dernière version "latest".
    ports:
      - "8080:8080"
    # Mappage du port hôte 8080 au port du conteneur 8080.
    networks:
      - name: netapp
    # Ajout du conteneur au réseau nommé "netapp".
    state: started
    # État souhaité du conteneur : "started" (démarré).
    recreate: yes
    # Recrée le conteneur si nécessaire (basé sur le nom du conteneur).
    pull: true
    # Télécharge la dernière version de l'image Docker si elle n'est pas déjà présente.
  become: yes
  # Utilise les privilèges de superutilisateur pour exécuter cette tâche.
```
