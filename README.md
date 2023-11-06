# TP1 Questions - HELET Edwin


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

# Tips questions

## Why should we run the container with a flag -e to give the environment variables?

Utiliser le drapeau -e permet de définir ces variables lors de l'exécution du conteneur.
Cela permet de personnaliser le comportement de l'application sans avoir à modifier son image.

## Why do we need a volume to be attached to our postgres container?

Il est important de conserver les données de la base de données de manière persistante, même si le conteneur est arrêté ou supprimé.
En attachant un volume au conteneur PostgreSQL, on s'assure que les données de la base de données sont conservées et peuvent être sauvegardées ou restaurées.

## Why do we need a reverse proxy?

Le reverse proxy permet de centraliser la gestion des demandes clients (par exemple sur un serveur httpd) et de les rediriger vers les serveurs appropriés (sur le port 8080) en fonction de règles de routage

## Why is docker-compose so important?

Le docker compose simplifie le déploiement d'applications contenant plusieurs services Docker,
en automatisant la création, la configuration et la liaison des conteneurs. C'est un outil essentiel pour la gestion d'applications multi-conteneurs.

## Why do we put our images into an online repo?

Placer des images de conteneur dans Docker Hub,
permet de les partager, de les distribuer et de les rendre accessibles à
d'autres utilisateurs ou membres de l'équipe directement en ligne.

# TP2 - Github Actions

## What is GitHub Actions?

estcontainers est une bibliothèque Java qui permet d'exécuter des conteneurs Docker pendant les tests. Dans cet exemple, nous utilisons le conteneur PostgreSQL pour attacher notre application lors des tests. Lorsque vous exécutez la commande "mvn clean verify", un conteneur Docker est lancé pendant l'exécution des tests, ce qui est très pratique pour effectuer des tests d'intégration.

## What are testcontainers?

Testcontainers est une bibliothèque Java qui permet d'exécuter des conteneurs Docker pendant les tests. Dans cet exemple, nous utilisons le conteneur PostgreSQL pour attacher notre application lors des tests. Lorsque vous exécutez la commande "mvn clean verify", un conteneur Docker est lancé pendant l'exécution des tests, ce qui est très pratique pour effectuer des tests d'intégration.

## What is the purpose of Continuous Delivery (CD) in the context of software development?

Continuous Delivery (CD) is a software development practice that focuses on automatically delivering code changes to production or a staging environment after they have been thoroughly tested. The goal of CD is to ensure that code can be reliably and consistently deployed to production with minimal manual intervention. It helps streamline the release process and improve software quality.

## How do you secure environment variables in GitHub Actions?

To secure environment variables in GitHub Actions, you can use GitHub Secrets. Secrets are encrypted variables that you can store in your GitHub repository. They are not exposed in the workflow logs, making them a secure way to store sensitive information like API keys or credentials.

## How do you build and push Docker images using GitHub Actions?

You can build and push Docker images in GitHub Actions by defining a job that uses a Docker action. In the job configuration, specify the Dockerfile and context for your Docker image, and provide the necessary credentials to log in to a container registry (e.g., Docker Hub). After building the image, you can use the action to push it to the container registry.

# TP 3 - Ansible


