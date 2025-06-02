# README: Dockerized Backend + MongoDB Setup for Library Management System

---

## Introduction

This document explains how to run your **Node.js backend** and **MongoDB database** using Docker containers. We will connect both containers using a **custom Docker network** so they can talk to each other properly. This setup is easy to use and follows industry best practices.

---

## Prerequisites

* Docker installed on your machine (Docker Desktop or Docker Engine)
* Basic knowledge of terminal/command prompt
* Your backend code is ready with proper MongoDB connection string support via environment variable `MONGO_URL`

---

## Step 1: Create a Custom Docker Network

We create a Docker network to connect backend and MongoDB containers. This network acts like a private local network where containers can communicate by their names.

```bash
docker network create -d bridge library-mgm
```

* `-d bridge`: Uses the default bridge driver, which is enough for local container networking.
* `library-mgm`: This is the name of the network.

---

## Step 2: Run MongoDB Container

We will start MongoDB in a container connected to our network.

```bash
docker run -d \
  --name library-mongo \
  --network library-mgm \
  -p 27017:27017 \
  -v mongo-data:/data/db \
  mongo:6
```

Explanation:

* `-d`: Run container in background.
* `--name library-mongo`: Name of this container is `library-mongo`.
* `--network library-mgm`: Connects container to our custom network.
* `-p 27017:27017`: Maps container’s MongoDB port 27017 to host port 27017 (to allow local access if needed).
* `-v mongo-data:/data/db`: Persists database data in Docker volume `mongo-data` to avoid data loss on container restart.
* `mongo:6`: Uses official MongoDB version 6 image.

---

## Step 3: Prepare Backend Code for MongoDB Connection

Make sure your backend code connects to MongoDB using the environment variable `MONGO_URL`.

Example connection code in Node.js (e.g., `conn.js` or `app.js`):

```js
const mongoose = require('mongoose');

mongoose.connect(process.env.MONGO_URL, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log("Connected to MongoDB"))
.catch(err => console.error("MongoDB connection error:", err));
```

**Important:**
Do NOT hardcode MongoDB connection string with `localhost`. Inside Docker, backend container will connect to MongoDB container by its **service/container name** `library-mongo`.

---

## Step 4: Build Backend Docker Image

Navigate to your backend folder (where Dockerfile exists) and build the image.

```bash
docker build -t library-mgm-backend .
```

This command creates a Docker image called `library-mgm-backend` based on your backend code and Dockerfile.

---

## Step 5: Run Backend Container

Run your backend container on the same Docker network and pass the MongoDB URL using environment variable.

```bash
docker run -d \
  --name library-backend \
  --network library-mgm \
  -p 8000:8000 \
  -e MONGO_URL=mongodb://library-mongo:27017/library \
  library-mgm-backend
```

Explanation:

* `-d`: Run container in background.
* `--name library-backend`: Container name.
* `--network library-mgm`: Connect backend to same Docker network as MongoDB.
* `-p 8000:8000`: Maps backend’s port 8000 to host’s port 8000.
* `-e MONGO_URL=...`: Passes MongoDB connection URL to backend inside container.
* `library-mgm-backend`: The backend image you built.

---

## Step 6: Test Everything

1. Check running containers:

```bash
docker ps
```

You should see `library-mongo` and `library-backend` running.

2. Check backend logs to verify MongoDB connection:

```bash
docker logs library-backend
```

Look for `Connected to MongoDB` message.

3. Open browser or Postman and hit:

```
http://localhost:8000
```

Your backend API should respond.

---

## Troubleshooting

* **Error: `ECONNREFUSED 127.0.0.1:27017`**

  This means your backend is trying to connect MongoDB on `localhost`. Inside Docker, `localhost` refers to backend container itself, not MongoDB. Always use `mongodb://library-mongo:27017` as the connection string when containers are on the same network.

* **MongoDB data not persisting**

  Make sure volume `mongo-data` is mounted properly. You can check volumes by:

  ```bash
  docker volume ls
  ```

* **Cannot connect to backend from host**

  Check port mapping and firewall settings.

---

## Summary

| Step           | Command / Info                                                                                                                                   |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Create network | `docker network create -d bridge library-mgm`                                                                                                    |
| Run MongoDB    | `docker run -d --name library-mongo --network library-mgm -p 27017:27017 -v mongo-data:/data/db mongo:6`                                         |
| Build backend  | `docker build -t library-mgm-backend .`                                                                                                          |
| Run backend    | `docker run -d --name library-backend --network library-mgm -p 8000:8000 -e MONGO_URL=mongodb://library-mongo:27017/library library-mgm-backend` |

---

## Notes

* Always rebuild backend image if you change code.
* Use environment variables for configuration; avoid hardcoding.
* Use Docker Compose in future for easier multi-container management.

---

**Happy Coding!**
If you need help with frontend containerization or full Docker Compose setup, just ask.

---

If you want, I can also help create a full `docker-compose.yml` for all your services in one file.


[Github](https://github.com/workwithshreesh)

