FROM node:22-alpine
WORKDIR frontend/
RUN npm install -g @angular/cli
COPY package*.json ./
RUN npm install --legacy-peer-deps
COPY . .
EXPOSE 4200
CMD ["ng", "serve", "--host", "0.0.0.0"]

