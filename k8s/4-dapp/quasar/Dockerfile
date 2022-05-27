# Develop stage
FROM node:lts-alpine as develop-stage
WORKDIR /src
COPY package*.json ./
RUN npm i -g @quasar/cli@latest
COPY .. .

# Local-deps
FROM develop-stage as local-deps-stage
RUN npm install

# Build stage
FROM local-deps-stage as build-stage-spa
RUN quasar build -m spa

# Production stage
FROM nginx:stable-alpine as production-stage-spa
COPY --from=build-stage-spa /src/dist/spa /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
