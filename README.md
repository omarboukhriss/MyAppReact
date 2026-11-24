# TP4 : Docker et Docker Compose — application MERN

Application : `Client` (React + Vite), `Server` (Express), `mongodb` (image officielle `mongo`).

## Structure

```
MyAppReact/
├── Client/   (Dockerfile, .dockerignore)
├── Server/   (Dockerfile, .dockerignore)
└── docker-compose.yml
```

## Étapes réalisées

### 1. Cloner le projet
```bash
git clone https://github.com/wiem-kb/MyAppReact.git
cd MyAppReact
```

### 2. Dockerfiles
**Server** (`Server/Dockerfile`) : image `node:lts-alpine`, `WORKDIR /app`, copie de `package*.json`,
`npm install`, copie du reste, `EXPOSE 9000`, `CMD ["npm", "start"]`.
La variable `MONGO_URI` est lue par `src/index.mjs` (connexion avec `mongoose`) ; elle est définie dans Docker Compose.

**Client** (`Client/Dockerfile`) : image `node:lts-alpine`, `WORKDIR /app`, copie de `package*.json`,
`npm install`, copie du reste, `npm run build`, installation de `serve`, `EXPOSE 3000`,
`CMD ["serve", "-s", "dist", "-l", "3000"]`.

Remarques :
- Vite génère le build dans `dist` (et non `build` comme Create React App).
- Le `package-lock.json` du client a été généré sous Windows et ne contient pas les binaires Linux
  de `rolldown-vite` : le build échoue dans Alpine (`Cannot find native binding`).
  Le Dockerfile supprime donc le lock avant `npm install`.

Construction des images :
```bash
docker build -t mern-server ./Server
docker build -t mern-client ./Client
```
(L'énoncé écrit `-t mern-server ./client` pour le client : c'est une coquille, le nom de l'image doit être `mern-client`.)

### 3. Réseau Docker
```bash
docker network create mern-network
```

### 4. MongoDB
```bash
docker run -d --name mongodb --network mern-network mongo
```

### 5. Serveur et client
```bash
docker run -d --name server --network mern-network -e MONGO_URI=mongodb://mongodb:27017/mern -p 9000:9000 mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
```

### 6. Docker Compose
Le fichier `docker-compose.yml` définit les services `mongodb`, `server` (dépend de `mongodb`,
`MONGO_URI=mongodb://mongodb:27017/mern`) et `client` (dépend de `server`), tous sur le réseau
`mern-network` (driver `bridge`).

```bash
# si les conteneurs des étapes 4-5 tournent encore, les supprimer d'abord
docker rm -f client server mongodb

docker compose up --build
```

## Vérification
- Client : http://localhost:3000 → affiche « API is running »
- API : http://localhost:9000/api/health → `{"message":"API is running"}`
- Logs serveur : `docker logs server` → « MongoDB connected »

Arrêt : `docker compose down`
