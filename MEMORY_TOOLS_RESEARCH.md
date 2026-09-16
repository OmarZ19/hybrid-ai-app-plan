# Research-Based Tool Integration Plan

## Scope

This document records open-source projects and patterns worth borrowing for the roleplay app. The goal is **not** to turn the application into a dependency bundle. Prefer small, local, optional adapters and preserve the app's simple SQLite-first backend.

Repository research found promising projects in four areas:

- roleplay and lorebook conventions from SillyTavern and related tools
- hierarchical and long-term memory ideas from HiGMem, HiMem, LycheeMem, Mem0, and Letta
- character state and affect ideas from Shikigami-Protocol and related companion projects
- evaluation and continuity testing from Roleplay-Eval

Project maturity, licensing, and APIs vary. Before copying code, verify the current license and maintenance status of each upstream repository.

## Recommended integrations by priority

### Priority 1: Implement directly in this app

These are high-value and low-complexity:

1. **SillyTavern-compatible character/lorebook import and export**
   - Preserve familiar character-card fields.
   - Support keyword keys, secondary keys, priorities, insertion order, depth, constant entries, and enabled flags.
   - Preserve unknown fields for round-trip compatibility.

2. **Hierarchical memory**
   - Recent scene memory
   - Event memories
   - Stable character/world facts
   - Relationship state
   - Rolling summaries
   - Unresolved threads

3. **Hybrid retrieval**
   - Exact/entity/keyword matching first
   - Optional embeddings second
   - Recency and importance re-ranking
   - Diversity limit to avoid injecting five near-duplicate memories

4. **Evidence links**
   - Every extracted memory must point to source messages.
   - The UI must show why a memory was recalled and whether it entered the prompt.

5. **Long-session evaluation harness**
   - Automatically run continuity scenarios.
   - Track fact recall, lore precision, prompt tokens, summary size, and latency.

### Priority 2: Optional adapters

These are useful, but should not be required for basic operation:

- Mem0 adapter for users who want an external memory backend
- Letta adapter for users who want Letta-managed agent state
- Qdrant/Chroma adapter for users who want an external vector store
- Graph memory adapter inspired by Graphiti/OpenMemory
- Emotion and relationship state adapter inspired by Shikigami-Protocol

### Priority 3: Research-only ideas

Use these as design references rather than direct dependencies:

- HiGMem-style event and summary hierarchy
- HiMem-style topic-aware segmentation and reconsolidation
- test-time personality/style separation from TTM
- Roleplay-Eval-style relationship and continuity benchmarks

## Proposed architecture

```text
Raw messages
   ↓
Event extractor
   ↓
Typed memory records + evidence links
   ├── scene memory: recent and exact
   ├── episodic memory: events and outcomes
   ├── semantic memory: stable facts and lore
   ├── relational memory: character relationships
   ├── state memory: location, inventory, injuries, time
   └── open threads: promises, mysteries, unfinished actions
   ↓
Hybrid retriever
   ↓
Context budget manager
   ↓
Character + lorebook + selected memories + summary + recent turns
   ↓
Chat model
```

## Memory selection algorithm

Use a deterministic, explainable pipeline:

1. Extract entities from the latest user message and recent assistant message.
2. Activate constant lorebook entries.
3. Activate keyword and secondary-key lore entries.
4. Search exact memory facts by entity/name.
5. Search optional vector index by semantic similarity.
6. Add unresolved threads related to the current entities.
7. Score candidates using:
   - direct entity match
   - lexical match
   - semantic similarity
   - importance
   - recency
   - relationship relevance
   - repetition penalty
8. Select until the memory token budget is reached.
9. Return a context manifest explaining every selected and omitted item.

Never let vector similarity alone decide what enters context.

## Roleplay-specific state

In addition to ordinary memories, maintain a compact `world_state` object:

```json
{
  "current_location": "The western watchtower",
  "time": "The night after the festival",
  "inventory": ["silver key", "damaged map"],
  "active_relationships": [
    {
      "character": "Mira",
      "toward_user": "guarded but trusting",
      "confidence": 0.82
    }
  ],
  "open_threads": [
    "Find who left the silver key"
  ]
}
```

The state must be editable and must retain evidence links. It is not a replacement for the transcript; it is a compact continuity aid.

## What to borrow from the discovered projects

### SillyTavern and roleplay tooling

Borrow:

- character-card field conventions
- lorebook entry semantics
- keyword/secondary-key activation
- depth and insertion-order controls
- branch/regenerate/edit workflow ideas

Do not make users install SillyTavern. Build a compatible importer/exporter and a cleaner native experience.

### HiGMem and HiMem concepts

Borrow:

- event-level memory instead of raw-message-only chunks
- hierarchical summaries
- topic-aware segmentation
- memory reconsolidation when new facts supersede old facts
- retrieval that returns a small number of high-value items

Implement a simple version first. Keep raw evidence so reconsolidation is reversible.

### Letta

Borrow:

- separation between working context and long-term memory
- explicit memory blocks
- persistence across sessions
- memory inspection and editing

Do not make Letta mandatory. It should be an optional provider because the app's default backend must remain simple.

### Mem0 / LycheeMem

Evaluate as optional adapters only. Useful concepts include:

- automatic fact extraction
- memory add/update/delete semantics
- deduplication
- structured memory lifecycle

The app's native memory layer should remain the default to avoid external service requirements and unnecessary context overhead.

### Shikigami-Protocol

Borrow selectively:

- explicit emotional/relationship state
- background reflection
- proactive continuity checks

These should be disabled by default except for a small relationship-state tracker, because proactive behavior can distract from roleplay and increase model calls.

### Roleplay-Eval

Use as inspiration for a local benchmark suite:

- character consistency
- memory recall
- relationship continuity
- temporal consistency
- response style adherence

Create local fixtures rather than requiring the upstream benchmark at runtime.

## Dependency policy

The app should work with:

- Python
- SQLite
- one local model runtime
- no hosted database
- no required vector database
- no required external memory service

Optional integrations must be behind interfaces:

```text
MemoryStore
  ├── SQLiteMemoryStore (default)
  ├── Mem0MemoryStore (optional)
  ├── LettaMemoryStore (optional)
  └── GraphMemoryStore (experimental)
```

The same interface should be used by retrieval, UI, export, and evaluation.

## Acceptance criteria

- [ ] A user can import a common roleplay character card without manual editing.
- [ ] Lorebooks support keys, priorities, insertion order, depth, and constant entries.
- [ ] A long conversation retains important events without injecting the whole transcript.
- [ ] Memory retrieval is hybrid and explainable.
- [ ] Contradictory facts can be superseded rather than duplicated forever.
- [ ] State and relationship changes are represented compactly.
- [ ] External memory systems are optional adapters, not core dependencies.
- [ ] The app can run fully offline with SQLite.
- [ ] The app has a continuity benchmark before claiming memory quality.
