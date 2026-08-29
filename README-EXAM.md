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

