# 🐳 MERN Docker App

> 🚀 A practical learning project to understand Docker by containerizing a full-stack MERN application with Docker Compose.

---

## 🧭 What I Built

This project contains:

- ⚛️ React + Vite — Frontend
- 🟢 Node.js + Express — Backend
- 🍃 MongoDB — Database
- 🖥️ Mongo Express — MongoDB GUI
- 🐳 Docker — Containerization
- 🧩 Docker Compose — Multi-container management
- 📦 Docker Hub — Image sharing

### 🏗️ Architecture

    Browser
       │
       ├── 🌐 localhost:3000
       │         ↓
       │      Frontend
       │
       └── 🌐 localhost:5000
                 ↓
              Backend
                 ↓
           🍃 MongoDB :27017
                 ↑
          🖥️ Mongo Express
           localhost:8081

---

## 📁 Project Structure

    mern-docker-app/
    │
    ├── backend/
    │   ├── .env
    │   ├── .dockerignore
    │   ├── Dockerfile
    │   ├── package.json
    │   └── index.js
    │
    ├── frontend/
    │   ├── .dockerignore
    │   ├── Dockerfile
    │   ├── package.json
    │   └── src/
    │       └── App.tsx
    │
    └── docker-compose.yml

---

# 🧠 1. Docker Basics

### 📦 Docker Image

An image is a blueprint used to create containers.

    Dockerfile
        ↓
    Docker Image
        ↓
    Container

### 📦 Docker Container

A container is a running instance of a Docker image.

### 📄 Dockerfile

A Dockerfile contains instructions used to build a Docker image.

### 🧩 Docker Compose

Docker Compose allows multiple containers to be defined and run together.

    Docker Compose
         │
         ├── ⚛️ Frontend
         ├── 🟢 Backend
         ├── 🍃 MongoDB
         └── 🖥️ Mongo Express

---

# 🐳 2. Backend Dockerfile

`backend/Dockerfile`

    FROM node:22.22.2

    WORKDIR /app

    COPY package*.json ./

    RUN npm install

    COPY . .

    EXPOSE 5000

    ENV CHOKIDAR_USEPOLLING=true
    ENV WATCHPACK_POLLING=true

    CMD ["npm", "run", "dev"]

### 🔍 Important Instructions

| Instruction | Meaning |
|---|---|
| `FROM` | Selects the base image |
| `WORKDIR` | Sets the working directory |
| `COPY` | Copies files into the image |
| `RUN` | Executes a command while building |
| `EXPOSE` | Documents the application port |
| `ENV` | Sets an environment variable |
| `CMD` | Command executed when the container starts |

---

# ⚛️ 3. Frontend Dockerfile

`frontend/Dockerfile`

    FROM node:20-alpine

    WORKDIR /app

    COPY package*.json ./

    RUN npm install

    COPY . .

    EXPOSE 5173

    ENV CHOKIDAR_USEPOLLING=true
    ENV WATCHPACK_POLLING=true
    ENV FAST_REFRESH=true

    CMD ["npm", "run", "dev", "--", "--host"]

Vite runs inside the container on:

    5173

I mapped it to my computer:

    localhost:3000

---

# 🚫 4. .dockerignore

`.dockerignore` prevents unnecessary files from being sent to the Docker build context.

Example:

    node_modules/
    dist/
    .env
    .env.local
    .env.*.local
    npm-debug.log*
    yarn-debug.log*
    yarn-error.log*
    pnpm-debug.log*
    .git/
    .gitignore
    .vscode/
    .idea/
    .DS_Store
    Thumbs.db
    Dockerfile
    .dockerignore

### 💡 Why?

I don't want unnecessary files such as:

    node_modules/
    dist/
    .git/
    .env

to be included in the Docker build context.

---

# 🧩 5. Docker Compose

`docker-compose.yml`

    services:

      mongo:
        image: mongo
        container_name: mongodbproject

        ports:
          - "27017:27017"

        environment:
          - MONGO_INITDB_ROOT_USERNAME=admin
          - MONGO_INITDB_ROOT_PASSWORD=password

        volumes:
          - mongo-data:/data/db


      backend:
        build: ./backend

        ports:
          - "5000:5000"

        env_file:
          - ./backend/.env

        volumes:
          - ./backend:/app
          - /app/node_modules

        environment:
          - CHOKIDAR_USEPOLLING=true
          - WATCHPACK_POLLING=true
          - NODE_ENV=development

        depends_on:
          - mongo

        restart: unless-stopped


      frontend:
        build: ./frontend

        ports:
          - "3000:5173"

        volumes:
          - ./frontend:/app
          - /app/node_modules

        environment:
          - CHOKIDAR_USEPOLLING=true
          - WATCHPACK_POLLING=true
          - FAST_REFRESH=true

        depends_on:
          - backend

        stdin_open: true
        tty: true

        restart: unless-stopped


      mongo-express:
        image: mongo-express

        restart: always

        ports:
          - "8081:8081"

        environment:
          - ME_CONFIG_MONGODB_URL=mongodb://admin:password@mongo:27017/?authSource=admin
          - ME_CONFIG_BASICAUTH_ENABLED=true
          - ME_CONFIG_BASICAUTH_USERNAME=admin
          - ME_CONFIG_BASICAUTH_PASSWORD=password

        depends_on:
          - mongo


    volumes:
      mongo-data:

---

# 🌐 6. Docker Networking

This was one of the most important concepts I learned.

## 💻 From My Computer

    localhost:3000  → Frontend
    localhost:5000  → Backend
    localhost:27017 → MongoDB
    localhost:8081  → Mongo Express

## 🐳 Inside Docker

Containers communicate using their **Docker Compose service names**.

    backend → mongo:27017

    mongo-express → mongo:27017

### ❌ Wrong

Inside the backend container:

    mongodb://localhost:27017

Why?

Because:

    localhost
       ↓
    means the backend container itself

### ✅ Correct

    mongodb://mongo:27017

Because:

    mongo
      ↓
    Docker Compose service name

---

# ⚠️ 7. Service Name vs Container Name

I used:

    mongo:
      container_name: mongodbproject

But the backend connects using:

    mongo

Not:

    mongodbproject

### 🧠 Remember

    Service name
         ↓
       mongo
         ↓
    Docker networking

`container_name` is not what I use for normal Compose service-to-service communication.

---

# 🔌 8. Port Mapping

Docker port format:

    HOST_PORT:CONTAINER_PORT

Example:

    ports:
      - "3000:5173"

Means:

    My Computer                  Container
    localhost:3000  ──────────→  5173

### 📌 My Port Mappings

    3000:5173    → ⚛️ Frontend
    5000:5000    → 🟢 Backend
    27017:27017  → 🍃 MongoDB
    8081:8081    → 🖥️ Mongo Express

---

# 🔄 9. Bind Mount

Backend:

    volumes:
      - ./backend:/app

Frontend:

    volumes:
      - ./frontend:/app

This connects my local source code with `/app` inside the container.

    Local Source Code
           ↓
       Bind Mount
           ↓
     Docker Container
           ↓
      Nodemon / Vite
           ↓
      Automatic Changes

Therefore, normal source-code changes don't require rebuilding.

---

# 📦 10. node_modules Volume

I used:

    - /app/node_modules

This keeps Docker's Linux `node_modules` separate from Windows `node_modules`.

    Windows node_modules
            ❌
            │
            X
            │
      Linux Container

    Docker node_modules
            ✅

This is useful when developing on Windows with Linux containers.

---

# 💾 11. MongoDB Volume

MongoDB stores database files inside:

    /data/db

I mounted:

    volumes:
      - mongo-data:/data/db

So:

    MongoDB Container
           ↓
        /data/db
           ↓
       mongo-data
           ↓
     Persistent Data

### ⚠️ Important

This command removes volumes:

    docker compose down -v

So it can remove the MongoDB data stored in the volume.

---

# 🔐 12. Environment Variables

I installed `dotenv`:

    npm install dotenv

Backend:

    require("dotenv").config();

    mongoose.connect(process.env.MONGODB_URI);

Created:

    backend/.env

Example:

    MONGODB_URI=mongodb://admin:password@mongo:27017/mydatabase?authSource=admin

Compose loads the environment file:

    backend:
      env_file:
        - ./backend/.env

### 💡 Why Environment Variables?

The code stays the same:

    mongoose.connect(process.env.MONGODB_URI);

Only the environment value changes.

    LOCAL
       ↓
    MONGODB_URI
       ↓
    Docker MongoDB


    PRODUCTION
       ↓
    MONGODB_URI
       ↓
    MongoDB Atlas

### 🔒 Never Push `.env`

Add this to `.gitignore`:

    .env

Never commit database passwords or other secrets to GitHub.

---

# 🔄 13. When Should I Rebuild?

## ❌ No Rebuild Usually Required

For normal code changes:

    index.js
    App.tsx
    CSS
    routes
    controllers
    middleware

Because source code is mounted using Docker volumes.

---

## ✅ Rebuild Required

### 📦 Installing a Package

Example:

    npm install dotenv

Then:

    docker compose up -d --build

Also rebuild when:

- `package.json` changes
- `package-lock.json` changes
- `Dockerfile` changes

### 🧩 Compose File Changes

For many `docker-compose.yml` changes:

    docker compose up -d

If the change requires rebuilding an image:

    docker compose up -d --build

---

# 🚀 14. Start the Project

From the project root:

    docker compose up -d --build

Check containers:

    docker compose ps

View logs:

    docker compose logs

Backend logs:

    docker compose logs backend

Follow backend logs:

    docker compose logs -f backend

---

# 🌍 15. Open the Application

### ⚛️ Frontend

    http://localhost:3000

### 🟢 Backend

    http://localhost:5000

### 🖥️ Mongo Express

    http://localhost:8081

### 🍃 MongoDB

    localhost:27017

---

# 🛠️ 16. General Docker Commands

## 🖼️ Images

List images:

    docker images

Build an image:

    docker build -t my-app .

Remove an image:

    docker rmi IMAGE_ID

Show image history:

    docker history IMAGE_NAME

---

## 📦 Containers

Running containers:

    docker ps

All containers:

    docker ps -a

Start:

    docker start CONTAINER_NAME

Stop:

    docker stop CONTAINER_NAME

Remove:

    docker rm CONTAINER_NAME

View logs:

    docker logs CONTAINER_NAME

Follow logs:

    docker logs -f CONTAINER_NAME

Enter a container:

    docker exec -it CONTAINER_NAME sh

---

# 🧩 17. Docker Compose Commands

Start:

    docker compose up

Start in background:

    docker compose up -d

Build + start:

    docker compose up -d --build

Stop containers:

    docker compose down

Stop + remove volumes:

    docker compose down -v

Check services:

    docker compose ps

View logs:

    docker compose logs

Backend logs:

    docker compose logs backend

Follow backend logs:

    docker compose logs -f backend

Build backend:

    docker compose build backend

Build frontend:

    docker compose build frontend

Restart backend:

    docker compose restart backend

---

# 🐳 18. Docker Hub

My Compose project creates separate images:

    mern-docker-app-backend
    mern-docker-app-frontend

MongoDB and Mongo Express already use public Docker images:

    mongo
    mongo-express

### 🏗️ Build Images

    docker compose build

Check images:

    docker images

### 🔑 Login

    docker login

### 🏷️ Tag Backend

    docker tag mern-docker-app-backend:latest ramitsonar/mern-docker-app-backend:0.0.2.RELEASE

### 🏷️ Tag Frontend

    docker tag mern-docker-app-frontend:latest ramitsonar/mern-docker-app-frontend:0.0.2.RELEASE

### ⬆️ Push Backend

    docker push ramitsonar/mern-docker-app-backend:0.0.2.RELEASE

### ⬆️ Push Frontend

    docker push ramitsonar/mern-docker-app-frontend:0.0.2.RELEASE

### 💡 Important

`docker tag` does not create another full copy of the image.

It gives the existing image another repository/tag name.

---

# 📦 19. My Docker Hub Images

### 🟢 Backend

    ramitsonar/mern-docker-app-backend:0.0.2.RELEASE

Pull:

    docker pull ramitsonar/mern-docker-app-backend:0.0.2.RELEASE

### ⚛️ Frontend

    ramitsonar/mern-docker-app-frontend:0.0.2.RELEASE

Pull:

    docker pull ramitsonar/mern-docker-app-frontend:0.0.2.RELEASE

---

# 🐛 20. Problems I Faced

## ❌ Docker Desktop Connection Error

Error:

    unable to get image...
    dockerDesktopLinuxEngine...
    The system cannot find the file specified

### Cause

Docker Desktop / Docker Linux engine was not running.

### Fix

Start Docker Desktop and run:

    docker compose up -d --build

---

## ❌ `users.map is not a function`

Backend returned:

    {
      message: "Users fetched successfully",
      users: []
    }

But frontend used:

    setUsers(res.data);

So `users` became an object instead of an array.

### ✅ Fix

    setUsers(res.data.users);

---

## ❌ MongoDB Authentication

After adding MongoDB username/password:

    mongodb://mongo:27017/mydatabase

was not enough.

### ✅ Correct

    mongodb://admin:password@mongo:27017/mydatabase?authSource=admin

---

## ❌ Docker Compose Indentation

Services must be directly under:

    services:

Correct:

    services:
      mongo:
      backend:
      frontend:
      mongo-express:

---

## ❌ Docker Push Syntax

Wrong:

    docker push ramitsonar/mern-docker-app-backend : 0.0.1.RELEASE

Correct:

    docker push ramitsonar/mern-docker-app-backend:0.0.1.RELEASE

There must be no spaces around `:`.

---

## ❌ Docker Hub `broken pipe`

If push fails with:

    broken pipe

it can be a network/upload interruption.

Retry:

    docker push ramitsonar/mern-docker-app-backend:0.0.1.RELEASE

Already uploaded layers can be reused.

---

# 🌐 21. Frontend API URL vs MongoDB URL

These are completely different URLs.

### ⚛️ Frontend → Backend

Local:

    VITE_API_URL=http://localhost:5000

Use:

    axios.get(`${import.meta.env.VITE_API_URL}/api/users`);

### 🟢 Backend → MongoDB

    MONGODB_URI=mongodb://admin:password@mongo:27017/mydatabase?authSource=admin

### 🧠 Remember

    Frontend
       ↓
    VITE_API_URL
       ↓
    Backend
       ↓
    MONGODB_URI
       ↓
    MongoDB

---

# 🚀 22. Production Deployment Idea

After development:

    React Frontend
          ↓
        Vercel

    Node + Express
          ↓
        Render
          ↓
      MongoDB Atlas

The backend code can remain:

    mongoose.connect(process.env.MONGODB_URI);

Only the environment variable changes.

    LOCAL
      ↓
    MONGODB_URI
      ↓
    Docker MongoDB


    PRODUCTION
      ↓
    MONGODB_URI
      ↓
    MongoDB Atlas

---

# 🧠 23. Final Mental Model

    🧩 Docker Compose
             │
       ┌─────┼─────┐
       │     │     │
       ▼     ▼     ▼
    ⚛️      🟢     🍃
    Frontend Backend MongoDB
     :5173    :5000   :27017
                │
                ▼
             mongo:27017

From my computer:

    localhost:3000  → ⚛️ Frontend
    localhost:5000  → 🟢 Backend
    localhost:27017 → 🍃 MongoDB
    localhost:8081  → 🖥️ Mongo Express

Inside Docker:

    backend → mongo:27017
    mongo-express → mongo:27017

---

# 🎯 24. Key Lessons

1. 🐳 **Dockerfile → Image → Container**
2. 🧩 Docker Compose manages multiple containers.
3. 🔌 `HOST:CONTAINER` is the port mapping format.
4. 🏠 `localhost` inside a container means that container itself.
5. 🌐 Containers communicate using Compose service names.
6. 🔄 Bind mounts allow development changes without rebuilding.
7. 📦 Dependency changes require rebuilding.
8. 💾 MongoDB volumes persist database data.
9. 🔐 `.env` keeps configuration outside the source code.
10. 🚫 Never push `.env` to GitHub.
11. 🌍 Frontend API URL and MongoDB URL are different.
12. 🏷️ `docker tag` gives an image a repository/tag name.
13. ⬆️ `docker push` uploads an image to Docker Hub.
14. ⚠️ `docker compose down -v` can delete database volumes.
15. 🔄 Local MongoDB and MongoDB Atlas can use the same code through environment variables.

---

# 🏁 Final Result

I successfully built and containerized a MERN application using:

    ⚛️ React/Vite
          +
    🟢 Node.js/Express
          +
    🍃 MongoDB
          +
    🖥️ Mongo Express
          +
    🐳 Docker
          +
    🧩 Docker Compose
          +
    📦 Docker Hub

> 💡 **Main goal:** Not just making the application run, but understanding how Docker images, containers, networking, volumes, environment variables, Docker Compose, and Docker Hub work together.