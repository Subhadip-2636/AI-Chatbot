# 🚀 Setup Guide — Groq AI Chatbot

A step-by-step guide to get the FastAPI (Python) backend and React (Vite) frontend chatbot running locally from scratch.

---

## 📋 Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Project Structure](#2-project-structure)
3. [Environment Variables](#3-environment-variables)
4. [Installation](#4-installation)
5. [Running the App](#5-running-the-app)
6. [Verifying It Works](#6-verifying-it-works)
7. [Common Issues & Fixes](#7-common-issues--fixes)
8. [Deployment](#8-deployment)

---

## 1. Prerequisites

Make sure you have the following installed and ready before starting:

| Requirement | Version | How to check | Where to get it |
|---|---|---|---|
| **Python** | 3.10 or higher | `python --version` or `python3 --version` | [python.org](https://www.python.org/) |
| **Node.js** | 18 or higher | `node -v` | [nodejs.org](https://nodejs.org) |
| **npm** | Comes with Node | `npm -v` | Bundled with Node.js |
| **Groq API Key** | Free tier available | — | [console.groq.com](https://console.groq.com) |

### Getting a Groq API Key (free, no credit card)

1. Go to [console.groq.com](https://console.groq.com) and sign up.
2. Navigate to **API Keys** in the left sidebar.
3. Click **Create API Key**, give it a name, and copy the key.
4. Store it somewhere safe — you won't be able to see it again.

---

## 2. Project Structure

The project has been split into a dedicated FastAPI backend and a Vite-based React frontend:

```
groq-chatbot/
├── backend/
│   ├── main.py            # FastAPI server + Groq API integration
│   ├── requirements.txt   # Python dependencies
│   ├── .env               # Your local secrets (never commit this!)
│   ├── .env.example       # Template — copy this to .env
│   └── .gitignore         # Excludes python virtualenv & .env
├── frontend/
│   ├── index.html         # Entry point for React app
│   ├── package.json       # Node dependencies & scripts
│   ├── vite.config.js     # Vite configuration (ports & proxy config)
│   └── src/
│       ├── main.jsx       # React entry point
│       ├── App.jsx        # Main React chat component
│       └── index.css      # Custom styling
├── README.md
└── setup.md               # ← You are here
```

---

## 3. Environment Variables

The backend reads configuration from a `.env` file in the [backend/](file:///c:/Users/ankur/Downloads/groq-chatbot/groq-chatbot/backend) folder.

### Step 1 — Copy the example file

From the project root directory, run:

```bash
cd backend
cp .env.example .env
```

On Windows (PowerShell):
```powershell
cd backend
Copy-Item .env.example .env
```

### Step 2 — Fill in your API key

Open [backend/.env](file:///c:/Users/ankur/Downloads/groq-chatbot/groq-chatbot/backend/.env) and set your key:

```env
# Required — get your free key at https://console.groq.com/keys
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Optional overrides (defaults shown):
# PORT=8000
# GROQ_MODEL=openai/gpt-oss-120b
```

### Available Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `GROQ_API_KEY` | ✅ Yes | — | Your Groq API key (`gsk_...`) |
| `PORT` | ❌ No | `8000` | Port the FastAPI server listens on |
| `GROQ_MODEL` | ❌ No | `openai/gpt-oss-120b` | Groq model to use |

> ⚠️ **Never commit `.env` to git.** It is listed in [backend/.gitignore](file:///c:/Users/ankur/Downloads/groq-chatbot/groq-chatbot/backend/.gitignore). Your API key must stay server-side — the browser never sees it.

---

## 4. Installation

### Step 1 — Install Backend Dependencies

It is highly recommended to use a Python virtual environment to manage dependencies.

1. Navigate to the `backend/` directory:
   ```bash
   cd backend
   ```
2. Create a virtual environment:
   ```bash
   python -m venv venv
   ```
3. Activate the virtual environment:
   - **Windows (Command Prompt):**
     ```cmd
     venv\Scripts\activate
     ```
   - **Windows (PowerShell):**
     ```powershell
     .\venv\Scripts\Activate.ps1
     ```
   - **macOS / Linux:**
     ```bash
     source venv/bin/activate
     ```
4. Install the Python packages:
   ```bash
   pip install -r requirements.txt
   ```

This installs the following packages defined in [requirements.txt](file:///c:/Users/ankur/Downloads/groq-chatbot/groq-chatbot/backend/requirements.txt):

| Package | Purpose |
|---|---|
| `fastapi` | Modern, fast web framework for building APIs with Python |
| `uvicorn` | ASGI server implementation to run the FastAPI app |
| `groq` | Official Groq API Python SDK with streaming support |
| `python-dotenv` | Loads `.env` variables into system environment variables |

### Step 2 — Install Frontend Dependencies

1. Navigate to the `frontend/` directory:
   ```bash
   cd ../frontend
   ```
2. Install Node.js dependencies:
   ```bash
   npm install
   ```

This installs React, Vite, and other configuration packages defined in [package.json](file:///c:/Users/ankur/Downloads/groq-chatbot/groq-chatbot/frontend/package.json).

---

## 5. Running the App

### Option A — Development Mode (Recommended)

Run the backend and frontend servers concurrently. This enables auto-reloads and fast refreshing on save for both Python files and React components.

1. **Terminal 1 — Start the Backend (FastAPI API):**
   ```bash
   cd backend
   # Make sure your virtual environment is active!
   python main.py
   ```
   *Expected backend output:*
   ```
   [OK] Groq chatbot server running at http://localhost:8000
      Using model: openai/gpt-oss-120b
   ```

2. **Terminal 2 — Start the Frontend (Vite Dev Server):**
   ```bash
   cd frontend
   npm run dev
   ```
   *Expected frontend output:*
   ```
     VITE v8.1.1  ready in X ms
     ➜  Local:   http://localhost:5173/
   ```

3. **Open the browser** and navigate to:
   ```
   http://localhost:5173
   ```
   *Note: Vite will automatically proxy all API calls (under `/api`) to the FastAPI backend running at `http://127.0.0.1:8000` via the configuration in [vite.config.js](file:///c:/Users/ankur/Downloads/groq-chatbot/groq-chatbot/frontend/vite.config.js).*

---

### Option B — Production / Build Mode

In production, you compile the React frontend into static assets and let FastAPI serve both the API and the user interface from a single port (`8000`).

1. **Build the frontend assets:**
   ```bash
   cd frontend
   npm run build
   ```
   This compiles your React project and outputs the files into the `frontend/dist` directory.

2. **Start the FastAPI backend:**
   ```bash
   cd ../backend
   # Make sure your virtual environment is active!
   python main.py
   ```
   FastAPI detects the `frontend/dist` directory and mounts it, serving the static frontend directly.

3. **Open the browser** and navigate to:
   ```
   http://localhost:8000
   ```

---

## 6. Verifying It Works

### Manual browser test
1. Open the Chat UI (either the Vite server at `http://localhost:5173` or the FastAPI server at `http://localhost:8000` depending on the mode).
2. Type **"Hello, who are you?"** — you should see a typing indicator, then streamed text appear word by word.
3. Follow up with **"What did I just ask you?"** — confirms conversation memory is working.
4. Click **Clear Chat** (or the reset option) — confirms history resets properly.
5. Refresh the page mid-conversation — confirms `localStorage` persistence.

### Health check (terminal)
Confirm the backend API is running and healthy:
```bash
curl http://localhost:8000/api/health
```

### API test with curl
Check the streaming API directly:
```bash
curl -N -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"Say hi in 5 words"}]}'
```
*The `-N` flag disables output buffering so you can watch tokens stream in real time.*

---

## 7. Common Issues & Fixes

| Symptom | Cause | Fix |
|---|---|---|
| `[ERROR] Missing GROQ_API_KEY` on startup | `.env` missing or key is empty | Run `cp .env.example .env` in the `backend/` folder and paste your key. |
| `ModuleNotFoundError` or `ImportError` on backend | Virtual environment not active or dependencies not installed | Run `venv\Scripts\activate` (or `source venv/bin/activate`) and run `pip install -r requirements.txt`. |
| `[WARNING] Frontend build not found at ...` | `npm run build` has not been run | For production mode, run `npm run build` in the `frontend/` folder. For development, run Vite dev server via `npm run dev`. |
| `401 Unauthorized` API error | Invalid or expired Groq API key | Regenerate a valid key at [console.groq.com/keys](https://console.groq.com/keys) and update `.env`. |
| `429 Too Many Requests` API error | Hit the client rate limit (20 req/min) or Groq's API limit | Wait a minute, or adjust `RATE_LIMIT` inside [main.py](file:///c:/Users/ankur/Downloads/groq-chatbot/groq-chatbot/backend/main.py#L75). |
| Chat bubbles fail to update or stay empty | Model name is deprecated or invalid | Verify active models at [console.groq.com/docs/models](https://console.groq.com/docs/models), and update `GROQ_MODEL` in `.env`. |
| Code changes in frontend/backend not updating | Running the wrong dev servers or cached build | In development, ensure you are running `npm run dev` in `frontend/` and opening `http://localhost:5173` rather than accessing build output. |

---

## 8. Deployment

The FastAPI backend can be deployed to any cloud provider supporting Python web applications.

### Option A — Render.com
1. Push the project repository to GitHub.
2. In the Render Dashboard, click **New** → **Web Service** and select your repository.
3. Set the following settings:
   - **Root Directory:** `backend`
   - **Runtime:** `Python`
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `python main.py`
4. Under **Environment Variables**, add:
   - `GROQ_API_KEY` = *your API key*
   - `PORT` = `8000` (Render will override or map this automatically)
5. Deploy. Render will generate a public HTTPS URL.

*Note: Since the backend expects a compiled frontend static build in `frontend/dist`, you may need to configure a pre-deployment step to run `npm run build` in the `frontend/` directory, or compile and check in the `frontend/dist` directory before pushing.*

### Option B — Railway.app
1. Push your repository to GitHub, and link it via **New Project** → **Deploy from GitHub repo** on Railway.
2. Set the root directory settings to `backend`.
3. Add the `GROQ_API_KEY` environment variable.
4. Railway will automatically detect the Python project and start it.
