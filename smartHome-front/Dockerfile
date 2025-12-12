# Étape 1: Construction avec Node.js
FROM node:14.15.0-alpine as builder
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

# Étape 2: Création de l'image finale avec Nginx
FROM nginx:alpine
COPY --from=builder /app/dist/smart-home /usr/share/nginx/html
