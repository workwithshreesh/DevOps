From node:12-alpine

WORKDIR /node

COPY . .

RUN npm install
RUN npm run test

EXPOSE 8000
CMD ["node","app.js"]



