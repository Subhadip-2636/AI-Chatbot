# Groq AI Chatbot — Documentation

A lightweight, full-stack AI chatbot powered by the [Groq API](https://console.groq.com). It has a Node.js/Express backend that streams responses from Groq's LLMs, and a vanilla HTML/CSS/JS frontend that renders the conversation in real time, ChatGPT-style.

---

## 1. Overview

| | |
|---|---|
| **Backend** | Node.js + Express |
| **AI Provider** | Groq API (`groq-sdk`) |
| **Frontend** | Vanilla HTML/CSS/JavaScript (no framework) |
| **Response mode** | Streaming (token-by-token) |
| **Persistence** | Browser `localStorage` (client-side only — no database) |
| **Default model** | `openai/gpt-oss-120b` |

The backend serves both the API and the static frontend files, so the whole app runs from a single server on one port.

---

## 2. Project Structure

```
groq-chatbot/
├── backend/
│   ├── server.js          # Express server + Groq API integration
│   ├── package.json       # Dependencies and npm scripts
│   ├── package-lock.json
│   ├── .env.example       # Template for required environment variables
│   └── .gitignore
├── frontend/
│   ├── index.html         # Chat UI markup
│   ├── style.css          # Styling
│   └── script.js          # Chat logic: sending messages, streaming, storage
└── README.md
```

---

## 3. How It Works

### Request flow

1. The user types a message in the browser and hits Enter/Send.
2. `script.js` adds the message to an in-memory `conversation` array and sends the **entire conversation history** to the backend via `POST /api/chat`.
3. `server.js` validates the request, prepends a system prompt, and forwards the messages to Groq's Chat Completions API with `stream: true`.
4. As Groq generates tokens, the backend immediately writes each token to the HTTP response (`res.write(token)`), so the client receives a live stream of plain text.
5. The frontend reads the response body with a `ReadableStream` reader and appends each chunk to the assistant's chat bubble as it arrives, producing the "typing" effect.
6. Once the stream ends, the full assistant reply is saved into the conversation array and persisted to `localStorage`.

### Why there's a backend at all

The Groq API key is a secret. If the frontend called Groq directly, the key would be visible in browser dev tools to anyone visiting the page. The backend keeps the key server-side and only ever exposes a generic `/api/chat` endpoint to the browser.

---

## 4. Backend (`backend/server.js`)

### Environment variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `GROQ_API_KEY` | Yes | — | Your Groq API key. Server refuses to start without it. |
| `PORT` | No | `3000` | Port the server listens on. |
| `GROQ_MODEL` | No | `openai/gpt-oss-120b` | Groq model ID to use. See [console.groq.com/docs/models](https://console.groq.com/docs/models) for current options. |

Copy `.env.example` to `.env` in the `backend/` folder and fill in your key before starting the server.

### Endpoints

#### `GET /api/health`
Simple liveness check. Returns:
```json
{ "status": "ok", "model": "openai/gpt-oss-120b" }
```

#### `POST /api/chat`
Sends a conversation to Groq and streams back the reply as plain text.

**Request body:**
```json
{
  "messages": [
    { "role": "user", "content": "Hello!" },
    { "role": "assistant", "content": "Hi there! How can I help?" },
    { "role": "user", "content": "What's the capital of France?" }
  ]
}
```

- `messages` must be a non-empty array.
- Each item needs `role` (`"user"` or `"assistant"`) and non-empty string `content`.
- Only the last 20 messages are used, to keep requests fast and within context limits.
- A fixed system prompt is prepended server-side to set the assistant's tone/behavior (edit `systemMessage` in `server.js` to customize).

**Success response:** `200 OK` with `Content-Type: text/plain`, streamed in chunks as the model generates tokens.

**Error responses:**
| Status | Cause |
|---|---|
| `400` | Missing/malformed `messages` array |
| `401` | Invalid Groq API key |
| `429` | Rate limit exceeded (see below) |
| `500` | Any other error contacting Groq |

### Rate limiting

A simple in-memory limiter caps each IP address to **20 requests per 60 seconds** (`RATE_LIMIT` / `RATE_WINDOW_MS` in `server.js`). This is meant to protect your Groq bill for small/demo deployments — it resets on server restart and doesn't share state across multiple server instances. For production traffic, swap in a package like `express-rate-limit` with a shared store (e.g., Redis).

### Model configuration

The model is set via `GROQ_MODEL` and defaults to `openai/gpt-oss-120b`. Other tunables live directly in the `/api/chat` handler:
- `temperature: 0.7` — creativity/randomness (0 = deterministic, 1+ = more random)
- `max_tokens: 1024` — cap on reply length

---

## 5. Frontend (`frontend/`)

### `index.html`
Defines the chat layout: header with a "Clear" button, a scrolling message window, an error banner, and a text input form.

### `script.js`
Handles all client-side behavior:
- **Sending messages** — `handleSend()` pushes the user's message to `conversation`, renders it, then POSTs to `/api/chat`.
- **Streaming render** — reads the response body chunk-by-chunk and updates the assistant's bubble live.
- **Enter to send** — `Enter` submits; `Shift+Enter` inserts a newline.
- **Auto-resizing textarea** — grows as the user types.
- **Conversation persistence** — the full conversation is saved to `localStorage` under the key `groq-chatbot-history` after every message, and reloaded on page load. Clearing the chat wipes this storage.
- **Error handling** — network/API errors remove the incomplete assistant bubble and show a dismissible error banner instead of leaving a broken message.

### `style.css`
Visual styling for the chat bubbles, typing indicator, input area, and error banner.

---

## 6. Setup & Running Locally

### Prerequisites
- Node.js (v18+ recommended, for native `fetch`/`--watch` support)
- A free Groq API key from [console.groq.com/keys](https://console.groq.com/keys)

### Steps

```bash
# 1. Move into the backend folder
cd backend

# 2. Install dependencies
npm install

# 3. Create your .env file
cp .env.example .env
# then open .env and paste your real GROQ_API_KEY

# 4. Start the server
npm start
# or, for auto-restart on file changes during development:
npm run dev
```

The app will be available at **http://localhost:3000** (or whatever `PORT` you set). The backend serves the frontend automatically — no separate frontend server is needed.

### Verifying it's running
```bash
curl http://localhost:3000/api/health
# {"status":"ok","model":"openai/gpt-oss-120b"}
```

---

## 7. Customization Guide

| Want to... | Change this |
|---|---|
| Use a different model | Set `GROQ_MODEL` in `.env`, or edit the default in `server.js` |
| Change the assistant's personality | Edit `systemMessage.content` in `server.js` |
| Adjust reply length | Change `max_tokens` in `server.js` |
| Adjust creativity | Change `temperature` in `server.js` (0–2 range) |
| Change rate limits | Edit `RATE_LIMIT` / `RATE_WINDOW_MS` in `server.js` |
| Restyle the UI | Edit `frontend/style.css` |
| Change how much history is sent per request | Edit the `.slice(-20)` call in `server.js` |

---

## 8. Known Limitations

- **No user accounts / multi-user support** — conversation history lives only in the browser's `localStorage`, per browser/device.
- **In-memory rate limiting** — resets on restart, not shared across multiple server instances/processes.
- **No database** — nothing is persisted server-side; restarting the server loses no data (there was none to begin with) but also means no chat analytics, transcripts, or backups exist.
- **Single system prompt** — not currently configurable per user/session without code changes.

---

## 9. Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Server exits immediately with `❌ Missing GROQ_API_KEY` | `.env` missing or empty | Copy `.env.example` to `.env` and add your key |
| `401 Invalid Groq API key` in browser | Wrong/revoked key, or key not loaded | Check the key at [console.groq.com/keys](https://console.groq.com/keys); confirm `.env` is in `backend/` and correctly named |
| `429 Too many requests` | Rate limit hit | Wait 60 seconds, or raise `RATE_LIMIT` in `server.js` |
| Blank/empty assistant reply | Network interruption or model returned nothing | Frontend surfaces this as an error banner automatically; just retry |
| Chat history disappears | Browser storage cleared, private/incognito mode, or different browser/device | Expected — history is stored in `localStorage`, which is local to one browser |

---

## 10. Security Notes

- Never commit your `.env` file — it's already excluded via `.gitignore`.
- Never expose `GROQ_API_KEY` to the frontend/browser.
- The rate limiter is a basic safeguard, not a substitute for authentication if you deploy this publicly — consider adding auth and a more robust rate-limiting solution before production use.
