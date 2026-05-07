# OpenCare

AI-powered medical self-diagnosis assistant. Patients describe symptoms and an LLM agent (with optional web search) helps identify potential causes.

Built as a database systems course project at the University of Idaho, Spring 2026.

---

## Architecture

```
┌────────────────────────────────────────────────────────────┐
│                      Browser (app/)                        │
│          React 19 + Vite + TailwindCSS v4                  │
└──────────────────────┬─────────────────────────────────────┘
                       │ HTTP  (VITE_BACKEND_URL)
┌──────────────────────▼──────────────────────────────────────┐
│                   server/ (Node.js)                         │
│     Express 5 · Better Auth · Gemini · Tavily · pg          │
│                                                             │
│  ┌─────────────┐  ┌────────────┐  ┌──────────────────────┐  │
│  │  PostgreSQL │  │   MinIO    │  │  python-server/      │  │
│  │  (pgvector) │  │  (S3-like) │  │  Flask · PyMuPDF     │  │
│  └─────────────┘  └────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

| Repo | Purpose | Port                    |
|---|---|-------------------------|
| `app/` | React frontend — all pages, UI components | 5173 (dev), 4174 (prod) |
| `server/` | Node.js API — auth, AI, storage, DB queries | 8080                    |
| `python-server/` | Flask microservice — PDF/image redaction via PyMuPDF | 5000                    |

**Data flow:**
1. The frontend calls `server/` for all business logic.
2. `server/` handles auth (Better Auth + PostgreSQL), stores files in MinIO, queries Gemini + Tavily for AI responses, and calls `python-server/` to redact sensitive terms from uploaded files before storing them.

---

## Repositories

This project uses Git submodules. Each directory is its own repo:

```
OpenCare Project/
├── app/            ← frontend submodule
├── server/         ← backend submodule
├── python-server/  ← redaction microservice submodule
└── docker-compose.yml
```

---

## Setup

### Prerequisites

-  npm v22 (frontend + backend)
- [Python 3.10+](https://python.org/) with `pip`
- [Docker](https://docker.com/) (for PostgreSQL + MinIO)

### 1. Clone with submodules

```bash
git clone --recurse-submodules <repo-url>
cd "OpenCare Project"
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

### 2. Start infrastructure

```bash
docker compose up -d
```

This starts PostgreSQL (port 5432) and MinIO (port 9000, console at 9001).

### 3. Apply the database schema

```bash
psql postgresql://postgres:postgres@localhost:5432/postgres -f server/db-migrations/generate-tables.sql
```

### 4. Create a MinIO bucket

Open the MinIO console at `http://localhost:9001` (user: `minioadmin`, password: `minioadmin`), create a bucket named `opencarebucket`, and set its access policy to **public**.

### 5. Configure environment variables

Copy `.env.example` for each service to `.env` and fill in your values:
See `.env.example` for all variables and descriptions.

### 6. Install dependencies and run
You can use package manager of your choice

**Frontend** (`app/`):

```bash
cd app
npm install
npm run dev
```

**Backend** (`server/`):
```bash
cd server
npm install
npm run dev
```

**Python server** (`python-server/`):
```bash
cd python-server
python -m venv .venv

# Windows: .venv\Scripts\activate
source .venv/bin/activate   

# Mac/Linux: .venv\Scripts\activate
. .venv/bin/activate        

pip install flask pymupdf
python run.py
```

The app is now available at `http://localhost:5173`.