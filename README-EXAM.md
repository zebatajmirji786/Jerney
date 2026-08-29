# 🛤️ Jerney — Containerized Blog Platform

A Gen-Z vibe blog platform built with a 3-tier architecture. Jerney allows users to create, edit, and delete blog posts and interact through comments.

This project was containerized and deployed on **AWS EC2 using Docker Compose** as part of the practical examination.

---

## ✨ Features

- 📝 Create blog posts
- ✏️ Edit existing posts
- 🗑️ Delete posts
- 💬 Add comments to posts
- 🎨 Gen-Z inspired UI
- 🌙 Dark mode / glass-morphism styling
- 🔍 View individual posts and comments

---

# 🏗️ System Architecture

Jerney follows a 3-tier architecture:

```text
                         INTERNET
                            │
                            │ HTTP :80
                            ▼
                  ┌─────────────────────┐
                  │      AWS EC2        │
                  │                     │
                  │   Security Group    │
                  │   HTTP :80          │
                  │   SSH :22 (My IP)   │
                  │                     │
                  │  ┌───────────────┐  │
                  │  │ Docker Compose│  │
                  │  │               │  │
                  │  │   Frontend    │  │
                  │  │ React + Nginx │  │
                  │  │      :80      │  │
                  │  │       │       │  │
                  │  │       │ /api  │  │
                  │  │       ▼       │  │
                  │  │    Backend    │  │
                  │  │ Node.js/      │  │
                  │  │ Express :5000 │  │
                  │  │       │       │  │
                  │  │       │ db    │  │
                  │  │       ▼       │  │
                  │  │  PostgreSQL   │  │
                  │  │     :5432     │  │
                  │  │       │       │  │
                  │  │       ▼       │  │
                  │  │ Named Volume  │  │
                  │  │postgres-data  │  │
                  │  └───────────────┘  │
                  │                     │
                  │ jerney-network      │
                  │ Docker bridge       │
                  └─────────────────────┘
```

### Service Communication

```text
Frontend
   │
   │ backend:5000
   ▼
Backend
   │
   │ db:5432
   ▼
PostgreSQL
```


Docker service names are used for communication instead of fixed container IP addresses.

---

## 🔌 Port Configuration

| Component | Container Port | Host Port | Exposure |
|---|---:|---:|---|
| Frontend / Nginx | 80 | 80 | Public |
| Backend / Node.js | 5000 | Not published | Internal |
| PostgreSQL | 5432 | Not published | Internal |

Only port `80` is published to the EC2 host.

The backend and PostgreSQL services communicate through the Docker network.

---

# 🐳 Docker Implementation

## 1. Backend Container

The backend is containerized using `node:20-alpine`.

### Backend Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev

COPY . .

EXPOSE 5000

USER node

CMD ["node", "src/index.js"]
```

### Backend Dockerization

The Dockerfile follows these steps:

```text
node:20-alpine
      ↓
WORKDIR /app
      ↓
COPY package*.json
      ↓
npm install --omit=dev
      ↓
COPY application source
      ↓
EXPOSE 5000
      ↓
USER node
      ↓
Start Node.js application
```

### Backend Optimizations

- Uses lightweight `node:20-alpine`
- Installs only production dependencies
- Uses `.dockerignore`
- Places dependency installation before application source copying
- Uses Docker layer caching
- Runs the application as the non-root `node` user
- Keeps port `5000` internal to the Docker network

---

## 2. Frontend Container

The frontend uses a **multi-stage Docker build**.

### Build Stage

```text
node:20-alpine
      ↓
Install dependencies
      ↓
npm run build
      ↓
dist/
```

### Runtime Stage

```text
nginx:1.27-alpine
      ↓
Copy generated dist/
      ↓
Nginx serves frontend
```

The Node.js build environment is not included in the final runtime image.

### Benefits

- Smaller runtime image
- Separate build and runtime environments
- Nginx serves the production frontend
- Node.js build dependencies are excluded from the final runtime image

---
 
# 📦 Docker `.dockerignore`

## Backend

The backend `.dockerignore` excludes unnecessary files from the Docker build context:

```text
node_modules
npm-debug.log
.git
.env
*.log
```

## Frontend

The frontend `.dockerignore` excludes:

```text
node_modules
dist
.git
.env
*.log
```

This reduces unnecessary files being sent to the Docker build process.

---

# ⚡ Docker Layer Caching

The backend Dockerfile was deliberately ordered so that dependency-related layers can be reused.

```text
WORKDIR
   ↓
COPY package*.json
   ↓
npm install --omit=dev
   ↓
COPY application source
```

When the application source changes but `package.json` remains unchanged, Docker can reuse the dependency installation layer.

### Verified Build Cache

Repeated builds showed:

```text
CACHED [2/5] WORKDIR /app
CACHED [3/5] COPY package*.json ./
CACHED [4/5] RUN npm install --omit=dev
CACHED [5/5] COPY . .
```

This confirmed that Docker layer caching was working.

---

# 📊 Docker Image Size

Image sizes were checked using:

```bash
docker image inspect jerney-backend --format='{{.Size}}'
docker image inspect jerney-frontend --format='{{.Size}}'
```

Measured sizes:

| Image | Approximate Size |
|---|---:|
| `jerney-backend` | 52.1 MB |
| `jerney-frontend` | 21.1 MB |

The image list was also checked with:

```bash
docker images | grep jerney
```

---

# 🐙 Docker Compose

Docker Compose is used to manage the complete application stack.

The Compose stack contains three services:

```text
frontend
backend
db
```

All three services are connected to:

```text
jerney-network
```

### Compose Architecture

```text
                 jerney-network
                       │
       ┌───────────────┼───────────────┐
       │               │               │
       ▼               ▼               ▼
   frontend         backend            db
   Nginx:80      Node.js:5000     PostgreSQL:5432
       │               │               │
       └─────── API ───┘               │
                       │               │
                       └──── DB ───────┘
```

---

# ❤️ PostgreSQL Healthcheck

The PostgreSQL service uses a healthcheck based on `pg_isready`.

```yaml
healthcheck:
  test:
    - CMD
    - /usr/bin/pg_isready
    - -U
    - postgres_user
    - -d
    - postgres
  timeout: 5s
  interval: 5s
  retries: 5
  start_period: 10s
```

The database was verified as:

```text
Up (healthy)
```

### Service Dependencies

The backend waits for PostgreSQL to become healthy:

```yaml
depends_on:
  db:
    condition: service_healthy
```

The frontend depends on the backend service:

```yaml
depends_on:
  backend:
    condition: service_started
```

Therefore:

```text
PostgreSQL
     ↓
Backend
     ↓
Frontend
```

---

# 🔐 Environment Variables

Database configuration was externalized instead of hard-coding the values in `docker-compose.yml`.

## Backend Variables

```text
DB_HOST
DB_NAME
DB_PASSWORD
DB_PORT
DB_USER
```

## PostgreSQL Variables

```text
POSTGRES_DB
POSTGRES_PASSWORD
POSTGRES_USER
```

The `.env` file is excluded from Git.

### Verify `.env` is Ignored

```bash
git check-ignore -v .env
```

The output confirmed:

```text
.gitignore:3:.env .env
```

### Verify `.env` Is Not Tracked

```bash
git ls-files .env
```

No output confirmed that `.env` is not tracked by Git.


> ⚠️ Never commit real passwords, credentials, or other secrets to Git.

---

# 🌐 Docker Networking

A custom Docker bridge network was created through Docker Compose:

```text
jerney-network
```

The network was inspected using:

```bash
docker network inspect jerney_jerney-network
```

The inspection verified that these containers were connected:

```text
jerney-frontend-1
jerney-backend-1
jerney-db-1
```


---

# 💾 PostgreSQL Persistent Storage

PostgreSQL uses a Docker named volume:

```text
jerney_postgres-data
```

The volume is mounted inside the database container at:

```text
/var/lib/postgresql/data
```

### Storage Flow

```text
PostgreSQL
     │
     ▼
/var/lib/postgresql/data
     │
     ▼
jerney_postgres-data
```

The volume was verified using:

```bash
docker inspect jerney-db-1
```

The mount showed:

```text
Name: jerney_postgres-data
Destination: /var/lib/postgresql/data
```

The volume was also listed using:

```bash
docker volume ls | grep jerney
```

Output:

```text
local     jerney_postgres-data
```

> ⚠️ `docker compose down -v` should not be used when the PostgreSQL data needs to be preserved because it removes the named volume.

---

