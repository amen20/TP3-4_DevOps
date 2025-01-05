# Guide de Configuration Docker

Ce guide détaille la configuration Docker pour notre application MERN stack.

## Structure des fichiers Docker

```
project/
├── client/
│   └── Dockerfile
├── server/
│   └── Dockerfile
└── docker-compose.yml
```

## Dockerfile du Client

```dockerfile
FROM node:lts-alpine

WORKDIR /usr/src/app

# Copier les fichiers package d'abord pour un meilleur cache
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le code source de l'application
COPY . .

# Construire l'application
RUN npm run build

# Installer serve
RUN npm install -g serve

# Exposer le port 3000
EXPOSE 3000

# Commande de démarrage
CMD ["serve", "-s", "build", "-l", "3000"]
```

Points clés :
- Utilise `node:lts-alpine` pour une image légère
- Copie les fichiers package séparément pour tirer parti du cache Docker
- Construit l'application React
- Utilise `serve` pour servir les fichiers statiques
- Expose le port 3000

## Dockerfile du Serveur

```dockerfile
FROM node:lts-alpine

WORKDIR /usr/src/app

# Copier les fichiers package d'abord pour un meilleur cache
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le code source de l'application
COPY . .

# Exposer le port 9000
EXPOSE 9000

# Commande de démarrage
CMD ["npm", "start"]
```

Points clés :
- Utilise la même image de base que le client
- Expose le port 9000 pour l'API
- Démarre le serveur avec npm start

## Configuration de Docker Compose

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo
    container_name: mongodb
    networks:
      - mern-network
    volumes:
      - mongodb_data:/data/db
    ports:
      - "27017:27017"

  server:
    build:
      context: ./server
      dockerfile: Dockerfile
    container_name: server
    ports:
      - "5000:5000"
    networks:
      - mern-network
    depends_on:
      - mongodb
    environment:
      - PORT=5000
      - MONGO_URI=mongodb://mongodb:27017/records
    volumes:
      - ./server:/usr/src/app
      - /usr/src/app/node_modules

  client:
    build:
      context: ./client
      dockerfile: Dockerfile
    container_name: client
    ports:
      - "3000:3000"
    networks:
      - mern-network
    depends_on:
      - server

volumes:
  mongodb_data:

networks:
  mern-network:
    driver: bridge
```

Points clés :
- Utilise l'image officielle de MongoDB
- Crée un volume persistant pour les données MongoDB
- Configure un réseau personnalisé pour la communication entre les conteneurs
- Les variables d'environnement sont configurées pour la connexion à la base de données
- Mappage des volumes pour un rechargement à chaud en développement
- Mappage des ports pour chaque service :
  - Client : 3000
  - Serveur : 5000
  - MongoDB : 27017

## Utilisation

1. Construire et démarrer les conteneurs :
```bash
docker-compose up --build
```

2. Arrêter les conteneurs :
```bash
docker-compose down
```

3. Voir les logs :
```bash
docker-compose logs -f
```

4. Accéder aux services :
- Frontend : http://localhost:3000
- API Backend : http://localhost:5000
- MongoDB : mongodb://localhost:27017
