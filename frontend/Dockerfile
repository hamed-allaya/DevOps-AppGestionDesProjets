# Étape 1 : Build Angular
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install --legacy-peer-deps
COPY . .
ENV NODE_OPTIONS="--max-old-space-size=1536"
RUN npm run build

# Étape 2 : Serveur Nginx
FROM nginx:alpine
COPY --from=build /app/dist/*/browser /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
