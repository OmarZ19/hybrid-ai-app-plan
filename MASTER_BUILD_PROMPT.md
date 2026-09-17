# Master Build Prompt: Roleplay-First Hybrid AI Desktop App

## Mission

Build a polished, local-first Windows desktop roleplay application that combines:

- Tauri + React + TypeScript desktop UI
- Python + FastAPI backend
- SQLite persistence
- local model providers such as Ollama, llama.cpp, or LM Studio
- a small reasoning model for planning, memory extraction, and continuity checks
- a larger chat model for final roleplay generation
- character cards and lorebooks
- dynamic cast detection and group responses
- structured User, Character, and Narrator turns
- layered long-term memory and summaries
- branching, checkpoints, rollback, and safe editing
- optional SkillOpt-based offline skill improvement

The result should feel like a calm, premium digital book or writing workspace, not a developer dashboard.

## Non-negotiable product rules

1. Roleplay is the primary use case.
2. Local-only operation must work with no cloud service or external database.
3. Character cards, lorebooks, conversations, memories, and settings are portable.
4. User, Character, and Narrator turns are distinct at the database, API, and UI layers.
5. The full transcript is preserved, but only relevant context is sent to the model.
6. Every memory and cast update has evidence links and can be edited or undone.
7. New characters can be detected automatically, but uncertain changes must remain provisional.
8. Multiple characters can respond in one generation without forcing every character to speak.
9. Narration must never be mixed into dialogue bubbles.
10. Optional features must have graceful fallbacks and clear controls.
11. Never overwrite a source character card automatically.
12. Never claim a feature is complete without running the specified tests.

## Recommended repository structure

```text
hybrid-ai-app/
├── backend/
│   ├── main.py
│   ├── config.py
│   ├── requirements.txt
│   ├── api/
│   │   ├── routes.py
│   │   ├── websocket.py
│   │   └── schemas.py
│   ├── database/
│   │   ├── models.py
│   │   ├── migrations/
│   │   └── db_init.py
│   ├── models/
│   │   ├── provider_base.py
│   │   ├── ollama_provider.py
│   │   ├── openai_compatible_provider.py
│   │   ├── reasoning_engine.py
│   │   ├── chat_engine.py
│   │   └── cascade_pipeline.py
│   ├── roleplay/
│   │   ├── cast_manager.py
│   │   ├── entity_resolver.py
│   │   ├── speaker_attribution.py
│   │   ├── group_response_planner.py
│   │   ├── turn_normalizer.py
│   │   ├── context_builder.py
│   │   ├── context_budget.py
│   │   ├── lorebook_engine.py
│   │   ├── memory_manager.py
│   │   ├── summary_manager.py
│   │   ├── world_state.py
│   │   ├── branch_manager.py
│   │   └── export_import.py
│   ├── skills/
│   │   ├── registry.py
│   │   ├── evaluator.py
│   │   └── skillopt_adapter.py
│   └── tests/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── UserTurn.tsx
│   │   │   ├── CharacterTurn.tsx
│   │   │   ├── NarratorTurn.tsx
│   │   │   ├── RoleplayTurnGroup.tsx
│   │   │   ├── CastPanel.tsx
│   │   │   ├── MemoryPanel.tsx
│   │   │   ├── ContextInspector.tsx
│   │   │   ├── LorebookPanel.tsx
│   │   │   ├── BranchPanel.tsx
│   │   │   └── SettingsDrawer.tsx
│   │   └── api/
│   └── tests/
├── tauri/
├── skills/
│   └── roleplay/
├── fixtures/
│   ├── character-cards/
│   ├── lorebooks/
│   └── long-session/
└── docs/
```

## Data model

Implement versioned migrations for at least:

- `characters`
- `character_aliases`
- `conversation_cast`
- `cast_events`
- `conversations`
- `messages`
- `roleplay_turns`
- `lorebooks`
- `lorebook_entries`
- `memory_entries`
- `memory_evidence`
- `summary_versions`
- `world_state_versions`
- `branches`
- `checkpoints`
- `generation_jobs`
- `context_manifests`
- `skill_versions`
- `evaluation_runs`

Important fields include stable IDs, revision IDs, schema versions, timestamps, status values, and source links. Use transactions for state-changing operations.

## Character cards and lorebooks

Support common Tavern/SillyTavern-style JSON and PNG metadata where legally and technically appropriate. Preserve unknown fields under `extensions` so imports can be exported without data loss.

Character cards must support:

- name
- description
- personality
- scenario
- first message
- alternate greetings
- example messages
- system prompt
- creator notes
- tags
- avatar metadata

Lorebooks must support:

- keys and secondary keys
- enabled/disabled state
- constant entries
- priority
- insertion order
- depth
- cooldown and use-once behavior
- character attachment
- global attachment

Validate imports before writing. Never execute imported content as application commands.

## Dynamic cast

Implement a Cast Manager that detects introductions such as:

- `called Sam`
- `named Sam`
- `this is Sam`
- `Sam walked in`
- `Sam: dialogue`

For a new character:

1. Create a session overlay.
2. Preserve the source message and evidence.
3. Add a provisional cast event when confidence is uncertain.
4. Add high-confidence characters automatically.
5. Give the character a stable ID and display name.
6. Derive initials and a deterministic avatar color by default.
7. Never invent personality or history.
8. Never mutate the imported global card unless the user selects `Save to card`.

Support aliases, rename, merge, split, remove from scene, reactivate, undo, and export.

## Structured roleplay turns

Canonical response data must use structured turns:

```json
{
  "turns": [
    {
      "type": "character",
      "speaker_id": "jade",
      "speaker_name": "Jade",
      "text": "You came back.",
      "sequence": 1
    },
    {
      "type": "narrator",
      "speaker_id": "narrator",
      "speaker_name": "Narrator",
      "text": "Jade lowers the lantern.",
      "sequence": 2,
      "follows_turn": 1,
      "related_speaker_ids": ["jade"],
      "scene_effect": false
    }
  ]
}
```

Supported types:

- `user`
- `character`
- `narrator`

Rules:

- Dialogue is character-only.
- Narration is prose, action, setting, transition, or consequence.
- Narration is visually separate from dialogue.
- A narrator block can be related to a character or the overall scene.
- Unknown speakers must be rejected or repaired.
- Store raw provider output for diagnostics, but render normalized turns.

## Group response planner

Before generation, select relevant speakers from the active cast using:

- current scene presence
- direct address
- character goals
- relationships
- recent speaker history
- user overrides
- maximum speaker setting

Default settings:

- multiple speakers enabled
- maximum character speakers: 2
- narrator enabled
- characters speak only when relevant
- no automatic all-cast responses

Support user overrides such as:

- `Jade responds`
- `Have Jade and Sam react`
- `Narrator only`
- `Everyone stays silent except Sam`

## Memory architecture

Use layered, evidence-based memory:

1. immutable app and roleplay rules
2. current scene and recent turns
3. active lorebook entries
4. retrieved memories
5. relationship and world state
6. structured rolling summaries
7. archived transcript available on demand

Memory types:

- fact
- event
- relationship
- preference
- world-state
- summary
- unresolved-thread
- character knowledge

Each memory requires:

- source turn IDs
- confidence
- importance
- lifecycle status
- created and updated timestamps
- optional character/entity links

When facts conflict, keep both evidence records, mark the older item superseded or stale, and allow user correction. Never silently delete evidence.

Use exact/entity/keyword retrieval by default. Embeddings, vector stores, graph memory, Mem0, Letta, or similar tools are optional adapters only.

## Summaries and context budgets

Preserve the complete transcript in SQLite. Build compact summaries containing:

- current scene
- character and relationship state
- important events and consequences
- world and inventory state
- unresolved threads
- stable facts

Summaries are versioned and never replace raw messages.

Before every request, produce a context manifest containing:

- configured token budget
- system tokens
- cast tokens
- recent-turn tokens
- lore tokens
- memory tokens
- summary tokens
- reserved output tokens
- omitted items and reasons

Never exceed the provider context limit.

## Streaming

Use WebSocket or Server-Sent Events with ordered events:

```text
generation_started
turn_started
turn_delta
turn_completed
cast_update
memory_update
summary_update
generation_completed
```

Partial turns are provisional until normalized and committed. Support cancellation, retries, job IDs, and one active generation per conversation by default.

## Editing, branches, checkpoints, and rollback

Editing an earlier turn must mark downstream turns, summaries, memories, cast events, and world state as stale or branch them away from the main line.

Implement:

- branch from any turn
- named checkpoints
- restore checkpoint
- archived side branches
- branch summaries
- conflict preview before merge
- safe regeneration from a selected point

A checkpoint includes turn sequence, cast, memories, summaries, lore usage, world state, and revision IDs.

## Provider abstraction

Implement a provider interface for:

- Ollama
- llama.cpp server
- LM Studio
- OpenAI-compatible endpoints
- optional hosted providers

Detect capabilities:

- structured output
- streaming
- context length
- stop generation
- health check
- model metadata

The app must still work when structured output is unavailable by using a constrained fallback parser and safe error state.

## SkillOpt integration

SkillOpt is an offline, opt-in skill compiler, not a live memory service and not part of the chat request path.

Use it to improve reusable procedures such as:

- memory extraction
- continuity checking
- summary compression
- relationship tracking
- lore selection
- speaker planning

Use versioned `current_skill.md`, `best_skill.md`, fixtures, scores, and history. Candidates must beat a held-out validation set before promotion. Provide dry-run, diff preview, rollback, cancellation, and clear unavailable-state behavior.

SkillOpt must never automatically rewrite raw transcripts, memories, character cards, safety settings, or user configuration.

## Privacy and safety

Defaults:

- local-only operation
- no telemetry
- no secrets in exports
- memory extraction can be disabled
- summaries can be disabled
- selective and full deletion controls
- imported story content is data, not system instructions

Prompt sections must be explicit:

```text
SYSTEM RULES
CHARACTER DEFINITIONS
LOREBOOK
MEMORY
WORLD STATE
CURRENT SCENE
USER INPUT
```

Imported content may not override application policy, user controls, speaker validation, or privacy settings.

## UI direction

Create a clean, book-like layout:

- left: library, characters, conversations, branches
- center: readable roleplay transcript
- right: cast, memory, lore, context inspector
- bottom: composer and compact model status

Every turn shows the correct speaker name and avatar. Narrator prose uses distinct typography. Add links for:

- memories
- cast events
- lore entries
- source evidence
- context inclusion reason

Basic users should not need to manage tokens, embeddings, or provider internals.

## Import/export and backups

Support:

- character cards
- PNG metadata cards
- lorebooks
- app-native structured conversations
- flattened compatibility exports
- memory bundles
- session cast overlays
- complete backups
- settings without secrets
- skill artifacts

Every backup has a schema version. Validate before restore, create a pre-migration backup, and show conflict previews.

## Quality dashboard

Add a debug/benchmark panel for:

- fact recall
- wrong-memory rate
- summary compression ratio
- token savings
- lore activation precision
- speaker attribution accuracy
- character consistency
- relationship consistency
- generation latency
- structured-output repair rate

Include fixtures that introduce Rum, Jade, and Sam, then test long-session recall after compression.

## Build order

1. Establish migrations, provider interfaces, and database transactions.
2. Implement character cards, lorebooks, and basic User/Character/Narrator rendering.
3. Implement dynamic cast and deterministic name detection.
4. Implement structured generation, normalization, and speaker validation.
5. Implement group response planning and ordered streaming.
6. Implement layered memory, evidence links, summaries, and context budgets.
7. Implement world state, relationship tracking, conflicts, and stale revisions.
8. Implement editing, branches, checkpoints, rollback, and backups.
9. Implement provider adapters and graceful fallback behavior.
10. Implement optional SkillOpt offline optimization.
11. Add quality dashboard and long-session tests.
12. Polish the UI, Windows packaging, documentation, and recovery behavior.

## Verification gate

Before declaring the app complete, run:

1. backend unit tests
2. API and database integration tests
3. provider mock tests
4. streaming cancellation tests
5. structured-output repair tests
6. Rum/Jade/Sam dynamic-cast scenario
7. Jade and Sam multi-speaker response scenario
8. narrator-separation rendering test
9. long-session memory and summary test
10. contradiction and supersession test
11. edit-and-regenerate test
12. branch and checkpoint restore test
13. import/export round-trip test
14. backup migration test
15. privacy and secret-exclusion test
16. manual Windows startup and packaged-build test

Do not claim that tests passed unless they were actually executed. Report any unavailable dependencies or skipped tests clearly.

## First implementation task

Start with Phase 1 only. Create the migration-safe backend foundation, provider interfaces, structured schemas, and test fixtures. Show the actual files and tests created. Do not implement the entire application in one speculative step.
