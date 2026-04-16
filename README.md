# Demo App — Developing with Docker

> A simple user profile application demonstrating a full local development setup using Docker, Docker Compose, Node.js, and MongoDB.

![Node.js](https://img.shields.io/badge/Node.js-Express-339933?logo=nodedotjs&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-6.0-47A248?logo=mongodb&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)

---

## Overview

This app demonstrates how to develop and run a multi-component application entirely with Docker. It consists of a static frontend, a Node.js backend, and a MongoDB database — each running as a container.

**Stack:**
- `index.html` — frontend with pure JavaScript and CSS
- `Node.js` + Express — backend server
- `MongoDB` — data storage
- `mongo-express` — browser-based MongoDB admin UI

---

## Running with Docker

Use this approach to run each container individually, giving you full control over the setup.

### Step 1 — Create a Docker network

```bash
docker network create mongo-network
```

> A custom network lets containers communicate by name. You can skip this step and omit `--net` from the commands below if you prefer to use the default Docker network.

### Step 2 — Start MongoDB

```bash
docker run -d \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  --name mongodb \
  --net mongo-network \
  mongo
```

### Step 3 — Start mongo-express

```bash
docker run -d \
  -p 8081:8081 \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
  -e ME_CONFIG_MONGODB_SERVER=mongodb \
  -e ME_CONFIG_MONGODB_URL=mongodb://mongodb:27017 \
  --name mongo-express \
  --net mongo-network \
  mongo-express
```

### Step 4 — Open mongo-express in the browser

```
http://localhost:8081
```

### Step 5 — Create the database and collection

In the mongo-express UI:
1. Create a new database called `user-account`
2. Inside it, create a collection called `users`

### Step 6 — Start the Node.js application

```bash
cd app
npm install
node server.js
```

### Step 7 — Open the application in the browser

```
http://localhost:3000
```

---

## Running with Docker Compose

Use this approach to start MongoDB and mongo-express together with a single command.

### Step 1 — Start MongoDB and mongo-express

```bash
docker-compose -f docker-compose.yaml up
```

mongo-express will be available at:
```
http://localhost:8080
```

### Step 2 — Create the database and collection

In the mongo-express UI:
1. Create a new database called `user-account`
2. Inside it, create a collection called `users`

### Step 3 — Start the Node.js application

```bash
cd app
npm install
node server.js
```

### Step 4 — Open the application in the browser

```
http://localhost:3000
```

---

## Building a Docker image

To package the application as a Docker image:

```bash
docker build -t my-app:1.0 .
```

> The `.` at the end tells Docker to look for the `Dockerfile` in the current directory.

---

## Pushing to a Nexus private registry

To store the image in a private Nexus repository, log in first, then tag and push:

```bash
docker login <nexus-host>:8083

docker tag my-app:1.0 <nexus-host>:8083/my-app:1.0

docker push <nexus-host>:8083/my-app:1.0
```

Replace `<nexus-host>` with the IP address or hostname of your Nexus server.

> For Nexus repository setup instructions, see the [Nexus documentation](https://help.sonatype.com/repomanager3).

---

## Deploying to a server

On the deployment server, log into the registry and start the application:

```bash
docker login <nexus-host>:8083

docker compose -f docker-compose.yaml up
```

---

## Project structure

```
.
├── app/
│   ├── server.js         # Node.js Express backend
│   ├── package.json
│   └── ...
├── docker-compose.yaml   # Compose config for MongoDB + mongo-express
├── Dockerfile            # Image build instructions for the app
└── index.html            # Frontend (pure JS + CSS)
```

---

## Ports reference

| Service | Port | URL |
|---|---|---|
| Node.js app | 3000 | http://localhost:3000 |
| mongo-express (Docker) | 8081 | http://localhost:8081 |
| mongo-express (Compose) | 8080 | http://localhost:8080 |
| MongoDB | 27017 | Internal only |
| Nexus registry | 8083 | `<nexus-host>:8083` |

---

## Potential improvements

- **Use environment variables for credentials** — move MongoDB username and password to a `.env` file and reference them in `docker-compose.yaml` rather than hardcoding them
- **Add a health check** — add `HEALTHCHECK` to the Dockerfile and `healthcheck` entries in Compose so Docker reports container readiness correctly
- **Push to ECR or DockerHub** — replace the Nexus registry with a cloud registry for easier team access
- **Add a `docker-compose.dev.yaml`** — separate development overrides (volume mounts, hot reload) from the base Compose file

---

## References

- [Node.js + Express documentation](https://expressjs.com/)
- [MongoDB Docker Hub image](https://hub.docker.com/_/mongo)
- [mongo-express Docker Hub image](https://hub.docker.com/_/mongo-express)
- [Docker Compose documentation](https://docs.docker.com/compose/)
