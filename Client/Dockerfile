FROM node:lts-alpine

WORKDIR /app

COPY package*.json ./
# Le package-lock.json vient de Windows : il manque les binaires Linux (rolldown).
# On le supprime pour que npm résolve les bonnes dépendances dans Alpine.
RUN rm -f package-lock.json && npm install

COPY . .

# Vite génère le build dans le dossier "dist"
RUN npm run build

# Serveur HTTP léger pour servir les fichiers statiques
RUN npm install -g serve

EXPOSE 3000

CMD ["serve", "-s", "dist", "-l", "3000"]
