# 🚀 How to Run the Project (Step-by-Step Guide)

This guide provides complete instructions to set up, configure, and run the **Multimodal RAG Pipeline by LangGraph** (FastAPI Backend + React Frontend + PostgreSQL + AI Services) locally or via Docker for development and hackathon presentations.

---

## 📋 Table of Contents
1. [Prerequisites](#1-prerequisites)
2. [Project Architecture Overview](#2-project-architecture-overview)
3. [Step 1: Environment Variables Configuration](#step-1-environment-variables-configuration)
4. [Step 2: Starting the Database (PostgreSQL + pgvector)](#step-2-starting-the-database-postgresql--pgvector)
5. [Step 3: Running the Backend (FastAPI + LangGraph)](#step-3-running-the-backend-fastapi--langgraph)
6. [Step 4: Running the Frontend (React + Vite)](#step-4-running-the-frontend-react--vite)
7. [Alternative: Running with Docker Compose](#alternative-running-with-docker-compose)
8. [Troubleshooting & Common Issues](#troubleshooting--common-issues)

---

## 1. Prerequisites

Make sure the following tools are installed on your machine:
- **Python**: `3.11` or `3.13` (Virtual environment `.venv` is already configured)
- **Node.js**: `v18+` or `v22+` (with `npm`)
- **Docker & Docker Compose**: (Optional, if running PostgreSQL locally)
- **Git**

---

## 2. Project Architecture Overview

| Component | Technology | Default Port / URL |
| :--- | :--- | :--- |
| **Backend API** | FastAPI, LangGraph, LangChain, PyTorch | `http://localhost:8000` |
| **Frontend UI** | React, Vite, TailwindCSS | `http://localhost:5173` |
| **Database** | PostgreSQL with `pgvector` extension | `localhost:5432` |
| **AI LLM / Tools** | Groq (Llama/Qwen/Whisper), Tavily Web Search, HuggingFace | Cloud APIs |

---

## Step 1: Environment Variables Configuration

### 1.1 Backend Environment File (`.env`)
In the **root** directory of the project, create a file named `.env` (or copy `.env.example` to `.env`):

```bash
cp .env.example .env
```

Open `.env` and fill in the required values:

```env
# ─── Required AI API Keys ───────────────────────────────
# Get free key from: https://console.groq.com/
GROQ_API_KEY=your_groq_api_key_here

# Get free key from: https://tavily.com/
TAVILY_API_KEY=your_tavily_api_key_here

# Get free access token from: https://huggingface.co/settings/tokens
HF_TOKEN=your_huggingface_token_here

# ─── Database ───────────────────────────────────────────
# Option A (Local Docker): postgresql://postgres:postgres@localhost:5432/zeus
# Option B (Neon Cloud):  postgresql://user:password@ep-xyz.neon.tech/zeus?sslmode=require
NEON_DATABASE_URL=postgresql://postgres:postgres@localhost:5432/zeus

# ─── Auth ───────────────────────────────────────────────
# Generate any secure random string or use a custom secret
JWT_SECRET_KEY=my_super_secret_hackathon_jwt_key_12345
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=10080

# ─── Deployment & CORS ──────────────────────────────────
ENVIRONMENT=development
CORS_ORIGINS=http://localhost:3000,http://localhost:5173

# ─── Optional Services ──────────────────────────────────
# Upstash Redis (Optional, leave blank to disable caching)
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=

# AWS S3 (Optional, falls back to local disk storage if blank)
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_STORAGE_BUCKET_NAME=
AWS_S3_REGION=us-east-1
```

---

### 1.2 Frontend Environment File (`frontend/.env`)
Inside the `frontend` directory, create a `.env` file to ensure API requests point to your local FastAPI backend:

```env
VITE_API_URL=http://localhost:8000
```

---

## Step 2: Starting the Database (PostgreSQL + pgvector)

The backend uses PostgreSQL with the `pgvector` extension for storing user state, documents, and LangGraph checkpointer history.

### Option A: Using Local Docker (Recommended)
Run the preconfigured `pgvector` container:
```powershell
docker compose -f docker-compose.local.yml up postgres -d
```
*Verify it is running on port `5432` with username `postgres`, password `postgres`, and database `zeus`.*

### Option B: Using Neon Cloud Database
1. Create a free PostgreSQL database on [Neon.tech](https://neon.tech).
2. Enable `pgvector` extension in Neon SQL editor: `CREATE EXTENSION IF NOT EXISTS vector;`.
3. Set `NEON_DATABASE_URL` in `.env` to your Neon connection string.

---

## Step 3: Running the Backend (FastAPI + LangGraph)

Open a terminal in the root project folder:

### 1. Activate the Python Virtual Environment
- **Windows (PowerShell)**:
  ```powershell
  .\.venv\Scripts\Activate.ps1
  ```
- **Windows (Command Prompt / CMD)**:
  ```cmd
  .\.venv\Scripts\activate.bat
  ```
- **Linux / macOS**:
  ```bash
  source .venv/bin/activate
  ```

*(If you ever need to re-install dependencies, run `uv pip install -r requirements.txt` or `pip install -r requirements.txt`)*

### 2. Start the FastAPI Server
```powershell
uvicorn backend.api.main:app --reload --port 8000
```

### 3. Verify Backend Status
- **Health Check**: Open [http://localhost:8000/health](http://localhost:8000/health) (should return `{"status": "ok", "version": "0.1.0"}`).
- **Interactive API Docs (Swagger)**: Open [http://localhost:8000/docs](http://localhost:8000/docs).

---

## Step 4: Running the Frontend (React + Vite)

Open a **second terminal** window:

### 1. Navigate to the `frontend` Directory
```powershell
cd frontend
```

### 2. (Optional) Install Frontend Dependencies
*(Dependencies are already installed, but if setting up on a new machine, run:)*
```powershell
npm install
```

### 3. Start the Vite Development Server
```powershell
npm run dev
```

### 4. Open in Browser
Visit **[http://localhost:5173](http://localhost:5173)** to access the Zeus AI Chatbot UI!

---

## Alternative: Running with Docker Compose

If you prefer to run the entire backend and database stack inside Docker:

```powershell
# Build and run backend + postgres services
docker compose -f docker-compose.local.yml up --build
```

Then run the frontend locally:
```powershell
cd frontend
npm run dev
```

---

## 🛠️ Troubleshooting & Common Issues

| Issue / Error | Cause | Solution |
| :--- | :--- | :--- |
| `[CONFIG ERROR] Missing required API keys` | One or more mandatory keys are missing from `.env` | Ensure `GROQ_API_KEY`, `TAVILY_API_KEY`, `HF_TOKEN`, `NEON_DATABASE_URL`, and `JWT_SECRET_KEY` are all filled in `.env`. |
| `Connection refused: localhost:5432` | PostgreSQL is not running | Start Postgres container with `docker compose -f docker-compose.local.yml up postgres -d` or check your database URL. |
| Frontend API calls failing (404/500/CORS) | Frontend pointing to old URL | Ensure `frontend/.env` has `VITE_API_URL=http://localhost:8000` and restart the Vite server (`npm run dev`). |
| Script Execution Policy Error in PowerShell | PowerShell blocking script activation | Run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` in PowerShell, then re-run `.\.venv\Scripts\Activate.ps1`. |

---

## 🎯 Ready for Hackathon Presentation!
1. Start Postgres: `docker compose -f docker-compose.local.yml up postgres -d`
2. Start Backend: `uvicorn backend.api.main:app --reload --port 8000`
3. Start Frontend: `cd frontend && npm run dev`
4. Open `http://localhost:5173` and start chatting, uploading PDFs, audio files, or images!
