# TP Questions - HELET Edwin

## Questions

1-1 Document your database container essentials: commands and Dockerfile:
    1. on utilise l'image postgres:14.1-alpine comme base. Alpine est une version légère de Linux, ce qui rend l'image plus petite et plus rapide.
    2. On définis des variables d'environnement pour le nom de la base de données, l'utilisateur et le mot de passe. Cela facilite la configuration initiale de PostgreSQL lors de la première exécution du conteneur.
    3. On copie deux scripts SQL dans un répertoire spécial (/docker-entrypoint-initdb.d/). Tout script dans ce répertoire sera automatiquement exécuté par PostgreSQL lors du démarrage du conteneur, ce qui permet d'initialiser la structure de la base de données et d'y insérer des données initiales.

1-2 Why do we need a multistage build? And explain each step of this dockerfile:
    --> Le build multi-étapes en Docker sert principalement à deux choses :
    1. Optimiser la taille de l'image finale : La première étape peut inclure tous les outils et fichiers nécessaires pour construire l'application (comme un JDK pour Java ou un compilateur pour C++), qui peuvent être volumineux. La seconde étape ne conserve que ce qui est nécessaire pour exécuter l'application (comme un JRE pour Java), réduisant ainsi la taille de l'image finale.
    2. Séparer les étapes de construction et d'exécution : Cela permet d'avoir une séparation claire entre la compilation de l'application et son exécution, rendant le Dockerfile plus propre et organisé.

--> Étape de construction (build) pour l'API backend:
`FROM maven:3.8.6-amazoncorretto-17 AS myapp-build`
Cette ligne indique qu'on utilise l'image `maven:3.8.6-amazoncorretto-17` comme base pour cette étape de construction. C'est une image contenant Maven pour la gestion de projets Java, ainsi que l'Amazon Corretto 17 JDK (une distribution de OpenJDK).
AS myapp-build donne un nom à cette étape, qui sera utilisé dans la suite du Dockerfile.

`ENV MYAPP_HOME /opt/myapp`
Cette ligne définit une variable d'environnement `MYAPP_HOME` avec la valeur `/opt/myapp`. Cette variable sera utilisée dans les étapes suivantes.

`WORKDIR $MYAPP_HOME`
Cette commande change le répertoire de travail dans l'image à `/opt/myapp` (la valeur de `MYAPP_HOME`).

`COPY pom.xml .`
Cette ligne copie le fichier `pom.xml` (qui est le fichier de configuration Maven de votre projet) du contexte de construction (généralement votre dossier local) vers le répertoire de travail dans l'image (c'est-à-dire `/opt/myapp`).

`COPY src ./src`
Cette commande copie le répertoire `src` (qui contient le code source de votre application) du contexte de construction vers le répertoire `/src` dans l'image

1.3 Document docker-compose most important commands:
`docker-compose up` : Construire et démarrer les services.
`docker-compose down` : Arrête et supprime les services.
`docker-compose build` : Construit ou reconstruit les services.
`docker-compose logs` : Affiche la sortie des conteneurs.
`docker-compose ps` : Liste les conteneurs.

1-4 Document your docker-compose file:
Pour `build` :, les chemins (`./backend`, `./httpd`) doivent pointer vers les répertoires contenant les fichiers Docker respectifs.
La directive `networks` : est utilisée pour connecter les services dans un réseau personnalisé pour la communication inter-conteneurs.

1-5 Document your publication commands and published images in dockerhub:
Après vous être connecté à Docker Hub avec `docker login`, marquez votre image :
$~docker tag my-database <USERNAME>/my-database:1.0
    Poussez votre image :
    $~docker push <USERNAME>/my-database:1.0

## Tips questions
- Why do we need a reverse proxy?