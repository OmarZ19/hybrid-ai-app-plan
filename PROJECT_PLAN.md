# PROJECT PLAN: Hybrid Reasoning + Chat AI Desktop App

## PROJECT OVERVIEW

Build a **Windows desktop application** that combines a small reasoning model with a larger chat model for efficient, intelligent conversations. The app will:
- Run **completely locally** on Windows
- Use **persistent memory** (SQLite)
- Orchestrate a **small reasoning model** → **large chat model** pipeline
- Provide a **clean, native Windows UI**
- Store conversation history and reasoning traces

**Tech Stack:**
- **Backend:** Python (FastAPI)
- **Frontend:** React + TypeScript
- **Desktop:** Tauri (Rust)
- **Models:** Qwen2.5-1.5B (reasoning), Llama-2-70B-Chat or Mistral-7B (chat)
- **Database:** SQLite
- **Model Runtime:** Ollama or llama.cpp

---

## PHASE 1: PROJECT SETUP & CORE INFRASTRUCTURE

### Goal 1.1: Initialize Project Structure
**Points: 3 | Time: 30-45 min**

Create a Windows-compatible project scaffold:

```
hybrid-ai-app/
├── backend/
│   ├── main.py                 # FastAPI server entry point
│   ├── models/
│   │   ├── reasoning_engine.py # Small model pipeline
│   │   ├── chat_engine.py      # Large model pipeline
│   │   └── cascade_pipeline.py # Orchestration logic
│   ├── database/
│   │   ├── models.py           # SQLAlchemy ORM models
│   │   ├── schema.sql          # Database schema
│   │   └── db_init.py          # Database initialization
│   ├── api/
│   │   ├── routes.py           # FastAPI routes
│   │   ├── schemas.py          # Pydantic request/response models
│   │   └── websocket.py        # WebSocket for real-time streaming
│   ├── config.py               # Configuration (model paths, DB, etc.)
│   ├── requirements.txt        # Python dependencies
│   └── .env.example            # Environment variables template
├── frontend/
│   ├── src/
│   │   ├── main.tsx            # React entry
│   │   ├── App.tsx             # Main component
│   │   ├── components/
│   │   │   ├── Chat.tsx        # Chat interface
│   │   │   ├── ReasoningView.tsx # Display reasoning trace
│   │   │   ├── ConvHistory.tsx # Conversation history sidebar
│   │   │   └── Settings.tsx    # Model/DB settings
│   │   ├── api/
│   │   │   └── client.ts       # HTTP/WS client for backend
│   │   ├── styles/
│   │   │   └── global.css
│   │   └── types/
│   │       └── index.ts        # TypeScript types
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
├── tauri/
│   ├── src/
│   │   └── main.rs             # Tauri entry point
│   ├── Cargo.toml
│   └── tauri.conf.json
├── docs/
│   └── SETUP.md                # Windows setup guide
├── .gitignore
└── README.md
```

**Deliverables:**
- [ ] Project folder structure created
- [ ] GitHub repo initialized (or local git)
- [ ] `.env.example` file with placeholders
- [ ] `README.md` with project overview

---

### Goal 1.2: Set Up Python Backend (FastAPI)
**Points: 4 | Time: 1-1.5h**

Create a minimal FastAPI server that will serve the pipeline.

**Key Files:**
- `backend/main.py` - FastAPI app
- `backend/requirements.txt` - Dependencies
- `backend/config.py` - Configuration

**Deliverables:**
- [ ] FastAPI app runs locally on `http://127.0.0.1:8000`
- [ ] `/health` endpoint returns `{"status": "ok"}`
- [ ] CORS configured
- [ ] Requirements.txt ready for `pip install`

---

### Goal 1.3: Set Up SQLite Database & ORM
**Points: 3 | Time: 45 min - 1h**

Create database models to store conversations and reasoning traces.

**Key Files:**
- `backend/database/models.py` - SQLAlchemy ORM models
- `backend/database/db_init.py` - Database initialization

**Tables:**
- `conversations` - Store conversation metadata
- `messages` - Store all messages (user, reasoning, assistant)
- `reasoning_traces` - Store detailed reasoning steps

**Deliverables:**
- [ ] SQLite database file created (`hybrid_ai.db`)
- [ ] Tables: `conversations`, `messages`, `reasoning_traces`
- [ ] ORM models work with SQLAlchemy
- [ ] `init_db()` creates schema on startup

---

## PHASE 2: MODEL INTEGRATION & PIPELINE

### Goal 2.1: Implement Small Reasoning Model Engine
**Points: 5 | Time: 1-1.5h**

Load and run the small reasoning model (Qwen2.5-1.5B).

**File: `backend/models/reasoning_engine.py`**

Key Requirements:
- Load model from Hugging Face (Qwen2.5-1.5B-Instruct)
- Support both GPU and CPU
- Generate structured reasoning output
- Handle long queries gracefully
- Return confidence scores

**Deliverables:**
- [ ] Reasoning model loads without errors
- [ ] `reason()` method returns structured output
- [ ] Works on both GPU and CPU
- [ ] Handles long queries gracefully
- [ ] Inference time < 10 seconds on GPU

---

### Goal 2.2: Implement Large Chat Model Engine
**Points: 5 | Time: 1-1.5h**

Load and run the large chat model (use Ollama for easier setup on Windows).

**File: `backend/models/chat_engine.py`**

Key Requirements:
- Connect to Ollama running locally
- Support model: Mistral-7B or Llama-2-70B-Chat
- Accept reasoning context as input
- Return conversational responses
- Handle errors gracefully

**Deliverables:**
- [ ] Chat engine connects to Ollama
- [ ] `chat()` method works with reasoning context
- [ ] Error handling for model unavailability
- [ ] Inference time < 20 seconds per response

---

### Goal 2.3: Implement Cascade Pipeline Orchestrator
**Points: 6 | Time: 1.5-2h**

Wire reasoning → chat in a coordinated pipeline.

**File: `backend/models/cascade_pipeline.py`**

Pipeline Flow:
1. User message → Reasoning model (step-by-step breakdown)
2. Reasoning + original message → Chat model (polished response)
3. Return both reasoning and response

**Deliverables:**
- [ ] Pipeline orchestrates reasoning → chat
- [ ] Returns structured JSON with both outputs
- [ ] Handles errors gracefully
- [ ] Conversation ID tracking
- [ ] Full pipeline completes in < 30 seconds

---

## PHASE 3: API ENDPOINTS & WEBSOCKET

### Goal 3.1: Implement REST API Endpoints
**Points: 4 | Time: 1-1.5h**

Create endpoints for chat interaction.

**File: `backend/api/routes.py`**

Endpoints:
- `POST /api/chat` - Send message, get reasoning + response
- `GET /api/conversations` - List all conversations
- `GET /api/conversations/{id}` - Get messages in conversation
- `DELETE /api/conversations/{id}` - Delete conversation

**Deliverables:**
- [ ] `POST /api/chat` endpoint works
- [ ] `GET /api/conversations` lists saved conversations
- [ ] `GET /api/conversations/{id}` retrieves messages
- [ ] `DELETE /api/conversations/{id}` removes conversation
- [ ] Database persistence works
- [ ] All responses are proper JSON

---

### Goal 3.2: Add WebSocket for Streaming (Optional)
**Points: 4 | Time: 1-1.5h**

Enable real-time response streaming via WebSocket.

**File: `backend/api/websocket.py`**

**Deliverables:**
- [ ] WebSocket endpoint at `/ws/chat`
- [ ] Streams reasoning first, then response
- [ ] Frontend can subscribe and receive updates in real-time
- [ ] Handles connection/disconnection gracefully

---

## PHASE 4: FRONTEND UI (React + Tauri)

### Goal 4.1: Set Up Tauri + React Project
**Points: 3 | Time: 45 min - 1h**

Initialize a Windows-native desktop app shell.

**Setup:**
```bash
cargo install tauri-cli
cargo tauri init
cd frontend && npm install
```

**Deliverables:**
- [ ] Tauri project scaffolded
- [ ] React development environment ready
- [ ] `npm run tauri dev` starts dev server
- [ ] Window opens on Windows

---

### Goal 4.2: Build Chat Interface Component
**Points: 5 | Time: 1.5-2h**

Create the main chat UI.

**File: `frontend/src/components/Chat.tsx`**

Features:
- Message display (user, assistant, reasoning roles)
- Input field for new messages
- Loading state during processing
- Auto-scroll to latest message
- Toggle to show/hide reasoning

**Deliverables:**
- [ ] Chat component renders messages
- [ ] Input field sends messages
- [ ] Loading state while waiting for response
- [ ] Toggle for reasoning view
- [ ] Clean, responsive styling

---

### Goal 4.3: Build Reasoning View Component
**Points: 3 | Time: 45 min - 1h**

Display reasoning in a collapsible sidebar.

**File: `frontend/src/components/ReasoningView.tsx`**

Features:
- Display reasoning trace
- Scrollable if long
- Visual distinction from main chat
- Optional collapse/expand

**Deliverables:**
- [ ] Reasoning sidebar displays and formats text
- [ ] Scrollable for long reasoning traces
- [ ] Clean visual hierarchy

---

### Goal 4.4: Build Conversation History Sidebar
**Points: 3 | Time: 45 min - 1h**

Allow users to save and load past conversations.

**File: `frontend/src/components/ConvHistory.tsx`**

Features:
- List all saved conversations
- Click to load conversation
- Delete button for each
- Highlight current conversation

**Deliverables:**
- [ ] Lists all saved conversations
- [ ] Click to load conversation
- [ ] Delete button for each
- [ ] Highlights current conversation

---

### Goal 4.5: Build Settings Panel
**Points: 3 | Time: 45 min - 1h**

Allow users to configure models and database.

**File: `frontend/src/components/Settings.tsx`**

Settings:
- Reasoning model selection
- Chat model selection
- Ollama base URL
- Database path
- All settings saved to localStorage

**Deliverables:**
- [ ] Settings panel renders
- [ ] Saves to localStorage
- [ ] Shows current config
- [ ] Changes persist across restart

---

## PHASE 5: DEPLOYMENT & TESTING

### Goal 5.1: Windows Executable Build
**Points: 4 | Time: 1-2h**

Package the app as a Windows .exe.

**Commands:**
```bash
# Build the Tauri app
cargo tauri build

# Output: src-tauri/target/release/Hybrid AI.exe
```

**Deliverables:**
- [ ] Builds to `.exe` file
- [ ] Runs standalone on Windows
- [ ] Creates installer (optional with WiX)
- [ ] App icon displays correctly

---

### Goal 5.2: End-to-End Testing
**Points: 4 | Time: 1.5-2h**

Test the full pipeline.

**Tests:**
- [ ] Backend starts without errors
- [ ] Models load in < 60 seconds
- [ ] Chat endpoint returns response in < 30 seconds
- [ ] UI renders all components
- [ ] Send message → Get reasoning + response
- [ ] Conversation history saves/loads
- [ ] Delete conversation works
- [ ] Settings persist across restart
- [ ] .exe builds and runs

---

### Goal 5.3: Documentation & Setup Guide
**Points: 3 | Time: 45 min - 1h**

Write Windows setup instructions.

**File: `docs/SETUP.md`**

Includes:
- Prerequisites (OS, hardware, software)
- Step-by-step installation
- Model download instructions
- Running the app
- Troubleshooting section
- Configuration options

**Deliverables:**
- [ ] Complete setup guide written
- [ ] Troubleshooting section
- [ ] Clear prerequisites listed
- [ ] Beginner-friendly instructions

---

## DELIVERABLES SUMMARY

### Phase 1 ✅
- [ ] Project structure created
- [ ] FastAPI backend running
- [ ] SQLite database initialized

### Phase 2 ✅
- [ ] Reasoning model loads and runs
- [ ] Chat model integrates (Ollama)
- [ ] Pipeline orchestrates both

### Phase 3 ✅
- [ ] REST API endpoints working
- [ ] Database persistence
- [ ] WebSocket streaming (optional)

### Phase 4 ✅
- [ ] Tauri + React setup
- [ ] Chat UI functional
- [ ] Reasoning view displays
- [ ] Conversation history sidebar
- [ ] Settings panel

### Phase 5 ✅
- [ ] Windows .exe builds
- [ ] All components tested
- [ ] Documentation complete

---

## ESTIMATED TIMELINE

- **Phase 1:** 2-4 hours
- **Phase 2:** 3-5 hours
- **Phase 3:** 3-4 hours
- **Phase 4:** 5-8 hours
- **Phase 5:** 2-3 hours

**Total: 15-24 hours** (or 2-3 days of focused work)

---

## KEY TECHNICAL DECISIONS

1. **Ollama for chat model** - Simplest Windows setup, handles model download/management
2. **SQLite for database** - No external DB dependency, works great on Windows
3. **Tauri + React** - Lightweight, native Windows feel, React ecosystem
4. **FastAPI backend** - Fast, async-friendly, great for LLM workflows
5. **Cascade pipeline** - Small model reasons, big model responds (efficient + high quality)

---

## NEXT STEPS

1. **Copy this entire file**
2. **Paste into ChatGPT** with prompt: "Build a Windows desktop app following this plan. Start with Phase 1. After completing each phase, show me the code and ask what's next."
3. **Work through each phase sequentially**
4. **Test after each phase**
5. **Commit code to git**
6. **Deploy to Windows .exe in Phase 5**

---

**Ready to build! 🚀**