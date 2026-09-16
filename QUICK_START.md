# Quick Start Guide

## For ChatGPT Users

### Step 1: Prepare
1. Open ChatGPT or your local AI
2. Copy the entire content from **PROJECT_PLAN.md**

### Step 2: Initial Prompt
Paste this into ChatGPT:

```
I want you to build a Windows desktop app that combines a small reasoning model 
with a larger chat model. Follow this detailed project plan:

[PASTE PROJECT_PLAN.md CONTENT HERE]

Start with Phase 1 - Project Setup & Core Infrastructure.

After you've completed Phase 1, show me all the code you've created and ask 
what the next step is. I'll then guide you to Phase 2.

Important:
- Create actual, working code (not pseudocode)
- Use the exact file paths and structure specified
- Show all necessary imports and dependencies
- Include error handling
```

### Step 3: Phase-by-Phase Workflow

For each phase:

1. **Ask ChatGPT:** "Complete Phase X, show me all the code"
2. **Review the code** ChatGPT provides
3. **Create files** locally with that code
4. **Test it:** Run the code, check for errors
5. **If it works:** Commit to git, move to next phase
6. **If it fails:** Share the error with ChatGPT, ask for fixes

### Step 4: Testing Commands

**Phase 1:**
```bash
cd backend
pip install -r requirements.txt
python main.py
# Should see: "INFO: Application startup complete"
```

**Phase 2:**
```bash
# Make sure Ollama is running: ollama serve
# In another terminal:
cd backend
python -c "from models.cascade_pipeline import get_pipeline; p = get_pipeline(); print(p.process('Hello'))"
```

**Phase 3:**
```bash
curl http://localhost:8000/health
# Should return: {"status":"ok"}
```

**Phase 4:**
```bash
cd frontend
npm install
npm run dev
```

**Phase 5:**
```bash
cargo tauri build
# Output: src-tauri/target/release/Hybrid AI.exe
```

---

## File Organization

After Phase 1, your structure should look like:

```
hybrid-ai-app/
├── backend/
│   ├── main.py
│   ├── requirements.txt
│   ├── models/
│   ├── database/
│   └── api/
├── frontend/
│   ├── package.json
│   └── src/
├── tauri/
├── .env
└── .gitignore
```

---

## Troubleshooting

### If Phase 1 fails:
- Python version issue? → Use Python 3.10+
- FastAPI won't start? → Check port 8000 is free
- Database error? → Delete `hybrid_ai.db`, re-run init

### If Phase 2 fails:
- Models won't load? → Make sure Ollama is running
- GPU memory error? → Use CPU mode (change config.py)
- Model download timeout? → Increase timeout in chat_engine.py

### If Phase 3 fails:
- Endpoint returns 500? → Check backend console for error
- Database not saving? → Verify SQLite file permissions
- CORS errors? → Ensure CORS is configured in main.py

### If Phase 4 fails:
- React won't compile? → Delete node_modules, npm install
- Tauri build fails? → Reinstall Rust via rustup
- API calls don't work? → Check backend is running on port 8000

### If Phase 5 fails:
- .exe won't build? → Check Rust installation
- .exe won't run? → Windows Defender or missing dependencies

---

## Tips for Success

1. **Test after each file created** - Don't wait until end of phase
2. **Keep terminal open** - See errors as they happen
3. **Use git commits** - `git add .` and `git commit -m "Phase X complete"`
4. **Save every response** - Copy ChatGPT code to files immediately
5. **Ask for clarification** - If something doesn't make sense, ask ChatGPT

---

## Expected Output

After all 5 phases:

✅ `hybrid-ai-app/` folder with complete source code  
✅ `backend/` running on localhost:8000  
✅ `frontend/` React app opens in Tauri window  
✅ Can chat with AI, see reasoning  
✅ Conversations saved to SQLite  
✅ `Hybrid AI.exe` file ready to share  

---

**You're ready! Start with Phase 1.** 🚀