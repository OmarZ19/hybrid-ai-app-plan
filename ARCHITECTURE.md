# System Architecture

## High-Level Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     WINDOWS DESKTOP APP                         │
│                      (Tauri + React UI)                         │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTP/WebSocket
                         ↓
┌─────────────────────────────────────────────────────────────────┐
│                   FastAPI Backend Server                        │
│                    (localhost:8000)                             │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              API Routes & Endpoints                     │  │
│  │  POST /api/chat - Send message & receive response      │  │
│  │  GET /api/conversations - List conversations           │  │
│  │  DELETE /api/conversations/{id} - Remove conversation  │  │
│  └────────────────────┬────────────────────────────────────┘  │
│                       │                                        │
│                       ↓                                        │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │         Cascade Pipeline Orchestrator                   │ │
│  │                                                          │ │
│  │  Input → Reasoning Engine → Chat Engine → Output        │ │
│  └─┬───────────────────────────────────────────────────────┘ │
│    │                                                         │
│    ├──→ Reasoning Model (Qwen 1.5B)                         │
│    │    - Step-by-step breakdown                           │
│    │    - Problem analysis                                  │
│    │    - Confidence scoring                               │
│    │                                                         │
│    └──→ Chat Model (Mistral/Llama via Ollama)              │
│         - Uses reasoning context                            │
│         - Generates conversational response                │
│         - Returns final answer                             │
│                                                             │
└──────────────────────┬───────────────────────────────────────┘
                       │
                       ↓
┌──────────────────────────────────────────────────────────────┐
│                   SQLite Database                            │
│                   (hybrid_ai.db)                            │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐ │
│  │Conversations │  │   Messages   │  │Reasoning Traces│ │
│  ├──────────────┤  ├──────────────┤  ├─────────────────┤ │
│  │id            │  │id            │  │id              │ │
│  │title         │  │conversation_ │  │message_id      │ │
│  │created_at    │  │id            │  │reasoning_steps │ │
│  │updated_at    │  │role          │  │confidence_     │ │
│  │              │  │content       │  │score           │ │
│  │              │  │timestamp     │  │timestamp       │ │
│  └──────────────┘  └──────────────┘  └─────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

---

## Component Breakdown

### Frontend (React + Tauri)

**Chat.tsx**
- Main chat interface
- Message display
- Input field
- Real-time updates

**ReasoningView.tsx**
- Displays reasoning trace
- Shows step-by-step breakdown
- Collapsible sidebar

**ConvHistory.tsx**
- Lists all conversations
- Load/delete functionality
- Current conversation highlight

**Settings.tsx**
- Model configuration
- Ollama URL setting
- Database path configuration

### Backend (FastAPI + Python)

**main.py**
- FastAPI application
- Server startup/shutdown
- Middleware configuration
- Route registration

**models/reasoning_engine.py**
- Loads Qwen2.5-1.5B model
- Generates reasoning traces
- Handles GPU/CPU switching
- Returns confidence scores

**models/chat_engine.py**
- Connects to Ollama service
- Sends requests to chat model
- Includes reasoning context
- Handles model timeouts/errors

**models/cascade_pipeline.py**
- Orchestrates reasoning → chat flow
- Combines outputs
- Manages conversation flow
- Tracks reasoning confidence

**api/routes.py**
- Implements all endpoints
- Request validation
- Response serialization
- Database operations

**database/models.py**
- SQLAlchemy ORM definitions
- Table schemas
- Relationships
- Query helpers

### Database (SQLite)

**conversations**
- Stores conversation metadata
- Tracks creation/update time
- Conversation title

**messages**
- Stores all messages in order
- Roles: user, reasoning, assistant
- Timestamps for each message

**reasoning_traces**
- Detailed reasoning output
- Confidence scores
- Links to message

---

## Data Flow Examples

### Example 1: Simple Chat Message

```
1. User types: "How do I optimize a Python loop?"
2. Clicks Send
3. Frontend sends: POST /api/chat with {message, conversation_id}
4. Backend receives request
5. Pipeline processes:
   a. Reasoning Engine analyzes query
      → Generates steps: "1) Identify bottleneck 2) Consider alternatives 3) Benchmark"
   b. Chat Engine takes reasoning + query
      → Generates: "Consider using list comprehensions instead of for loops..."
6. Backend returns JSON:
   {
     "conversation_id": "uuid",
     "user_message": "How do I optimize a Python loop?",
     "reasoning": "1) Identify bottleneck... 2) Consider alternatives...",
     "response": "Consider using list comprehensions...",
     "model_reasoning": "Qwen/Qwen2.5-1.5B-Instruct",
     "model_chat": "mistral"
   }
7. Frontend displays:
   - User message in blue bubble
   - Optional: Reasoning in yellow sidebar
   - Response in green bubble
8. Backend saves to database:
   - Conversation (if new)
   - User message
   - Reasoning message
   - Assistant response
   - Reasoning trace
```

### Example 2: Load Conversation History

```
1. User clicks "Previous Conversation" in sidebar
2. Frontend calls: GET /api/conversations/{conversation_id}
3. Backend queries messages from SQLite
4. Returns all messages in conversation
5. Frontend renders full conversation history
6. User can continue chatting in this conversation
```

### Example 3: Change Settings

```
1. User opens Settings panel
2. Changes "Chat Model" to "neural-chat"
3. Frontend saves to localStorage
4. On next app restart:
   a. Frontend loads settings from localStorage
   b. Sends to backend via environment/config
   c. Backend loads specified model from Ollama
   d. Next chat uses new model
```

---

## Technology Choices

### Why Qwen2.5-1.5B for Reasoning?
- ✅ Small enough to run on consumer hardware
- ✅ Fast reasoning generation (< 10 seconds)
- ✅ Excellent at step-by-step problem solving
- ✅ Multi-lingual support
- ✅ Open source on Hugging Face

### Why Mistral/Llama for Chat?
- ✅ High-quality natural language responses
- ✅ Good context understanding
- ✅ Supports function calling (future feature)
- ✅ Widely tested and reliable
- ✅ Available via Ollama for easy setup

### Why Ollama for Model Runtime?
- ✅ Simple Windows installation
- ✅ Automatic model download/caching
- ✅ GPU acceleration support
- ✅ REST API interface (easy to call)
- ✅ Doesn't require complex setup

### Why FastAPI?
- ✅ Async support (good for concurrent requests)
- ✅ Automatic API documentation
- ✅ Fast and efficient
- ✅ Built-in Pydantic validation
- ✅ Great for LLM applications

### Why Tauri + React?
- ✅ Cross-platform (Windows, Mac, Linux possible)
- ✅ Small bundle size
- ✅ Native Windows look and feel
- ✅ React ecosystem mature
- ✅ Easy to develop and iterate

### Why SQLite?
- ✅ No external database needed
- ✅ Single file (easy to backup)
- ✅ Good performance for this use case
- ✅ Works great on Windows
- ✅ No server to run

---

## Performance Considerations

### Reasoning Model
- **Model Size:** 1.5B parameters
- **Memory:** ~3-4 GB with fp16
- **Typical Time:** 5-10 seconds per query
- **Batch Size:** 1 (no batching needed)

### Chat Model
- **Model Size:** 7B-70B parameters (depending on choice)
- **Memory:** 14GB+ recommended for 7B, 40GB+ for 70B
- **Typical Time:** 10-20 seconds per query
- **Batch Size:** 1

### Database
- **Query Time:** < 50ms for typical queries
- **Storage:** ~100KB-1MB per conversation
- **No indexing needed** for initial implementation

### API
- **Throughput:** ~1 request per 30 seconds
- **Latency:** 20-40 seconds per full pipeline
- **Bottleneck:** Model inference, not API

---

## Security Considerations

- ✅ All processing local (no cloud calls)
- ✅ SQLite database not encrypted (offline use)
- ✅ No API authentication needed (localhost only)
- ✅ No sensitive data in logs
- ✅ Models run with user permissions

---

## Future Enhancements

1. **Multi-turn context** - Keep reasoning model aware of conversation history
2. **Tool use** - Let reasoning model call tools (calculator, search, etc.)
3. **Fine-tuning** - Train models on user-specific data
4. **Voice input** - Add speech-to-text support
5. **Export** - Save conversations as PDF/Markdown
6. **Cloud sync** - Optional sync to cloud backup
7. **Plugins** - Third-party model/tool integrations

---

**Architecture is clean, modular, and ready for implementation!** ✅