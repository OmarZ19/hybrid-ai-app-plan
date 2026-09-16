# Updated Build Instructions: Roleplay-First Implementation Order

Give the following instructions to the coding agent after it reads the repository plan.

## Build order

### Phase 1: Core roleplay data
Build the SQLite schema and services for:

- characters
- lorebooks
- lorebook entries
- conversations
- messages
- memory entries
- summary versions
- context manifests
- imported/exported assets

Do not begin with generic assistant settings. Roleplay data is the foundation.

### Phase 2: Character cards and lorebooks
Implement:

- import character-card JSON
- export character-card JSON
- drag-and-drop import
- character editor
- lorebook editor
- character-to-lorebook attachment
- unknown-field preservation

Add fixtures for at least one character card and one lorebook.

### Phase 3: Memory pipeline
Implement:

- automatic memory extraction
- fact/event/relationship/world-state categories
- source-message links
- relevance retrieval
- manual memory editing
- memory pinning
- memory rejection
- memory merge and supersession

Use simple lexical/entity matching first. Make embeddings optional.

### Phase 4: Context compression
Implement:

- recent-message window
- rolling structured summary
- summary versioning
- summary validation
- context budget manager
- context manifest returned by the API
- archived transcript retrieval

The app must preserve raw messages even after compression.

### Phase 5: UI
Build the UI as a clean writing workspace:

- library/sidebar for characters and conversations
- central chat page
- collapsible memory/lore/context panel
- memory links beside messages
- context inspector
- import/export actions
- minimal settings drawer

### Phase 6: Hybrid models and SkillOpt
Add:

- optional small reasoning model
- large chat model
- optional SkillOpt integration for evolving reusable roleplay skills
- model fallback behavior
- clear status and error states

The basic roleplay app must still work if the reasoning model, embeddings, or SkillOpt are disabled.

## Defaults

Enable by default:

- persistent conversations
- automatic memory extraction
- memory retrieval
- structured summaries
- lorebook keyword activation
- context budget management
- import/export

Make optional:

- embeddings
- SkillOpt training
- visible reasoning traces
- cloud providers
- background optimization
- multi-agent features

## Definition of done

Do not call the project complete until the roleplay memory test plan passes. Demonstrate a long session where important facts are recalled after compression, while the prompt remains within the configured budget.
