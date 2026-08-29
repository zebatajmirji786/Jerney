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

 
