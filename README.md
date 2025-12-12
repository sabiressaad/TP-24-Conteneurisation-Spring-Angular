# TP 24 : Conteneurisation Spring Angular

**Cours : Architecture Microservices : Conception, Déploiement et Orchestration**

## Structure du Projet

```
tp24/
├── .git
├── .idea
├── Smart_Home_back/       # Backend Spring Boot avec MySQL
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
├── smartHome-front/       # Frontend Angular
│   ├── src/
│   ├── package.json
│   └── Dockerfile
├── .env
└── docker-compose.yml
```

## Étapes de Conteneurisation

### Étape 1 : Organiser la structure du projet

1. Cloner les projets depuis https://github.com/lachgar/smarthouse.git
2. Assurer que le projet est organisé correctement avec deux dossiers principaux :
   - `Smart_Home_back` (backend)
   - `smartHome-front` (frontend)

### Étape 2 : Créer les Dockerfiles

#### Dockerfile Backend (Smart_Home_back/Dockerfile)

```dockerfile
# Stage 1: Build with Maven
FROM maven:3.8.4-openjdk-17 AS builder
WORKDIR /app
COPY ./src ./src
COPY ./pom.xml .
RUN mvn clean package

# Stage 2: Create the final image
FROM openjdk:17-jdk-alpine
VOLUME /tmp
ARG JAR_FILE=target/*.jar
COPY --from=builder /app/${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

**Explication :**
- **Étape 1** : Utilise Maven 3.8.4 avec OpenJDK 17 pour construire le package JAR
- **Étape 2** : Utilise Alpine Linux avec OpenJDK 17 pour créer une image légère et exécuter l'application

#### Dockerfile Frontend (smartHome-front/Dockerfile)

```dockerfile
# Étape 1: Construction avec Node.js
FROM node:14.15.0-alpine as builder
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

# Étape 2: Création de l'image finale avec Nginx
FROM nginx:alpine
COPY --from=builder /app/dist/smart-home /usr/share/nginx/html
```

**Explication :**
- **Étape 1** : Utilise Node.js 14.15.0 pour installer les dépendances et construire l'application
- **Étape 2** : Utilise Nginx pour servir les fichiers statiques de l'application Angular

### Étape 3 : Construire les images Docker

Le fichier `docker-compose.yml` à la racine définit 4 services :

1. **mysql** : Base de données MySQL
2. **backend** : Application Spring Boot
3. **frontend** : Application Angular
4. **phpmyadmin** : Interface web pour gérer MySQL

```bash
docker-compose build
```

### Étape 4 : Lancer les conteneurs

```bash
docker-compose up
```

Ajouter l'option `-d` pour exécuter en arrière-plan :

```bash
docker-compose up -d
```

### Étape 5 : Vérifier l'état des conteneurs

```bash
docker-compose ps
```

### Étape 6 : Accéder à l'application

- **Frontend** : http://localhost
- **Backend** : http://localhost:8085
- **PHPMyAdmin** : http://localhost:8081
- **MySQL** : localhost:3306

## Détails des Services

### MySQL
- Image : `mysql:latest`
- Port : 3306
- Base de données : `smart-house`
- Utilisateur : `root`
- Mot de passe : `root`

### Backend
- Build : `./Smart_Home_back`
- Port : 8085
- Dépend de : `mysql`
- Configuration : Connexion automatique à MySQL

### Frontend
- Build : `./smartHome-front`
- Port : 80
- Dépend de : `backend`

### PHPMyAdmin
- Image : `phpmyadmin/phpmyadmin`
- Port : 8081
- Connexion à MySQL automatique

## Commandes Utiles

```bash
# Construire les images
docker-compose build

# Lancer les conteneurs
docker-compose up -d

# Arrêter les conteneurs
docker-compose down

# Voir les logs
docker-compose logs -f

# Voir l'état des conteneurs
docker-compose ps

# Redémarrer un service spécifique
docker-compose restart backend

# Reconstruire et relancer
docker-compose up -d --build
```

## Notes Importantes

- Assurer que Docker et Docker Compose sont installés sur votre système
- Les ports 80, 3306, 8081 et 8085 doivent être disponibles
- Les fichiers de configuration Spring Boot doivent être configurés pour utiliser les variables d'environnement
- Le healthcheck vérifie que MySQL est prêt avant de lancer le backend

## Dépendances

- Docker
- Docker Compose
- Ports disponibles : 80, 3306, 8081, 8085
