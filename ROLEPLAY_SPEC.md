# Roleplay-First Product Specification

## Product direction

This app is a local-first, desktop roleplay client. It should combine the best parts of character-card clients, lorebook systems, long-term memory, and a clean writing environment.

The default experience must work immediately, while advanced features remain optional and discoverable.

## Non-negotiable priorities

1. **Roleplay quality over generic assistant behavior**
2. **Preserve long-term continuity with minimal context usage**
3. **Character cards and lorebooks are first-class objects**
4. **Memory is visible, editable, linked to messages, and exportable**
5. **Every major feature can be disabled without breaking basic chat**
6. **Local-first privacy and portable data**

---

## Feature policy: optional versus default

### Enabled by default

- Character-card loading
- Character persona and scenario prompts
- Conversation persistence
- Recent-message window
- Automatic memory extraction
- Memory retrieval for each turn
- Rolling conversation summary
- Lorebook keyword matching
- Import/export of conversations, characters, lorebooks, and settings
- Memory links shown in the message UI
- Context budget indicator
- Automatic backup of important memory updates

### Optional, disabled or conservative by default

- SkillOpt skill evolution/training
- Embedding-based semantic retrieval if it requires a separate service
- Hidden chain-of-thought display
- Automatic deletion or aggressive forgetting
- Background model optimization
- Cloud APIs
- Multi-agent orchestration
- Experimental reranking
- Automatic rewriting of user-authored character data

Every optional feature needs a toggle, a short explanation, and a safe fallback.

---

## Roleplay data model

### Character cards
Support importing and exporting common character-card formats, especially JSON character cards used by roleplay clients.

Store at minimum:

- `name`
- `description`
- `personality`
- `scenario`
- `first_mes`
- `mes_example`
- `system_prompt`
- `alternate_greetings`
- `creator_notes`
- `tags`
- `creator`
- `character_version`
- `extensions`
- `avatar_path` or embedded avatar metadata when supported

Do not discard unknown fields during import. Preserve them under `extensions` so cards can be round-tripped without data loss.

### Lorebooks
A lorebook contains entries that are conditionally injected into context.

Each entry should support:

- `id`
- `name`
- `keys`
- `secondary_keys`
- `content`
- `priority`
- `insertion_order`
- `enabled`
- `case_sensitive`
- `constant`
- `cooldown`
- `depth`
- `use_once`
- `tags`
- `character_id`
- `extensions`

Support both global lorebooks and character-attached lorebooks.

### Memory entries
Memory is separate from raw chat history.

Each memory should contain:

- `id`
- `conversation_id`
- `character_id`
- `type`: fact, event, relationship, preference, world-state, summary, unresolved-thread
- `content`
- `source_message_ids`
- `importance`
- `confidence`
- `created_at`
- `last_recalled_at`
- `last_confirmed_at`
- `status`: active, stale, superseded, rejected
- `embedding` or retrieval metadata, when enabled

Source links are essential: the user must be able to see which messages support a memory.

---

## Context-saving memory architecture

The app must not send the complete chat transcript on every turn. Use a layered context system.

### Layer 0: immutable instructions
Always include:

- safety and formatting rules
- character identity rules
- current mode settings

Keep this compact and cached where possible.

### Layer 1: current scene
Always include:

- the latest user message
- the last 4-12 roleplay turns, configurable by context budget
- the current scene/state block
- the immediately previous assistant response when needed

### Layer 2: active lore
Include only lorebook entries activated by:

- keyword matches
- character/location/entity mentions
- current scene metadata
- explicit user request

Rank entries by priority, recency, and relevance. Cap the number of inserted entries.

### Layer 3: retrieved memories
Retrieve only memories relevant to the current message. Combine:

- lexical matching for names and exact terms
- semantic retrieval when embeddings are enabled
- entity matching
- relationship and continuity relevance

Do not retrieve memories solely because they are recent.

### Layer 4: compressed history
Maintain a rolling summary of older conversation segments. The summary must preserve:

- character relationships
- major events and consequences
- promises and unresolved threads
- injuries, possessions, locations, discoveries, and world changes
- user preferences and boundaries
- exact names, dates, numbers, and important wording when relevant

Keep a structured state section separate from prose summary so facts are easier to preserve.

### Layer 5: archived transcript
Store the entire transcript locally, but do not inject it by default. Allow manual search and “bring into context.”

---

## Summarization policy

Use a small model or configured summarizer for compression. Never blindly overwrite the only copy of history.

For every compression operation:

1. Preserve the original messages in SQLite.
2. Generate a candidate summary.
3. Extract structured facts and unresolved threads.
4. Validate that names, entities, events, and state changes survived.
5. Save the summary as a versioned artifact.
6. Keep the previous summary until the new one is accepted.
7. Make the summary visible and editable.

Summaries should be cumulative but not recursively degrade indefinitely. Periodically rebuild from source messages or checkpoint summaries.

### Recommended summary format

```text
## Current scene
...

## Character and relationship state
- ...

## Important events and consequences
- ...

## World and inventory state
- ...

## Unresolved threads
- ...

## Stable facts
- ...
```

Avoid vague summaries such as “the characters discussed their past.” Preserve the concrete facts needed to continue the scene.

---

## Context budget manager

Before each model request, create a context manifest:

```json
{
  "budget": 12000,
  "system_tokens": 800,
  "recent_tokens": 4200,
  "lore_tokens": 900,
  "memory_tokens": 1300,
  "summary_tokens": 1800,
  "reserved_output_tokens": 3000,
  "dropped_items": []
}
```

The budget manager must:

- estimate tokens before sending
- reserve room for the model response
- preserve system and character identity first
- preserve current-scene messages next
- select lore and memories by relevance
- compress or omit low-value context
- show the user what was included when inspection is enabled

Never silently exceed the model context limit.

---

## Memory links in the UI

Every memory shown in the interface must link back to evidence.

### Message-level links
Add a small `Memory` or bookmark icon beside messages that produced or updated memories. Clicking it opens:

- memories extracted from the message
- source excerpt
- memory type
- confidence and importance
- edit, reject, merge, or delete controls

### Memory panel
The right-side panel should have tabs:

- `Relevant now`
- `Conversation memories`
- `Character facts`
- `World state`
- `Open threads`
- `Summary`

Each memory card should show:

- short content
- source message link
- why it was retrieved
- whether it was injected into the current prompt
- edit and pin controls

### Context inspector
Include a collapsible inspector showing:

- current token estimate
- active character card sections
- activated lorebook entries
- retrieved memories
- summary version
- omitted or trimmed items

This makes memory behavior understandable instead of magical.

---

## Import and export requirements

### Import
Provide file pickers and drag-and-drop for:

- character-card JSON
- character-card PNG metadata when supported
- lorebook JSON
- conversation JSON/JSONL
- app configuration JSON
- memory bundles
- SkillOpt Markdown skill files

Validate imports before writing data. Show a preview and preserve unknown fields.

### Export
Allow exporting:

- current character card
- character plus attached lorebook
- conversation transcript
- conversation with memory bundle
- all app data as a portable backup
- settings and model configuration without secrets
- optimized skill files

Exports must be documented and human-readable where possible. Never include API keys or passwords.

---

## Roleplay controls

Add a simple advanced-settings drawer with:

- response style and length
- temperature/top-p when supported
- context budget
- recent-turn count
- memory retrieval on/off
- lorebook on/off
- automatic summary on/off
- show/hide reasoning
- regenerate and branch conversation
- edit message
- continue response
- stop generation
- preserve exact character voice toggle

Basic users should not need to open this drawer.

---

## Backend design

Keep the backend small and explicit:

```text
request
  → load character
  → load conversation state
  → extract candidate memories from prior turn
  → activate lorebook entries
  → retrieve relevant memories
  → assemble and budget context
  → call reasoning/planning model if enabled
  → call chat model
  → save messages and memory candidates
  → update summary asynchronously
  → return response plus context manifest
```

The response API should return:

- assistant message
- conversation ID
- memory IDs created or updated
- activated lorebook entry IDs
- summary version
- context manifest
- model and timing metadata

Memory extraction and summarization can run after the visible response so they do not block normal chat, but they must be durable and retryable.

---

## Acceptance criteria

- [ ] User can import a character card and start roleplay immediately.
- [ ] User can attach or import a lorebook.
- [ ] Character, scenario, greeting, and examples affect the prompt correctly.
- [ ] The app saves the full transcript without sending it all every turn.
- [ ] The app creates editable memories with source-message links.
- [ ] Relevant memories are injected automatically and irrelevant memories are omitted.
- [ ] Older scenes are compressed into versioned summaries.
- [ ] Context budget is visible and never exceeded.
- [ ] User can inspect and edit active context.
- [ ] Character cards, lorebooks, conversations, memories, and configs export cleanly.
- [ ] Optional systems can be disabled independently.
- [ ] Default roleplay flow works without requiring database administration or manual prompt engineering.
- [ ] UI feels like a clean digital book or writing workspace, not an admin dashboard.
