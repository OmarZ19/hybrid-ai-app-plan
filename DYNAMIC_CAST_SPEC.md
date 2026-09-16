# Dynamic Cast and Speaker Identity System

## Purpose

Add a roleplay-aware **Cast Manager** that detects newly introduced people, places, and entities in the story, keeps their identities consistent, and updates the visible chat group automatically.

This is a core roleplay service, not a general-purpose autonomous agent. It should run as a small structured extraction step after each user and assistant message, with optional model assistance and deterministic validation.

## Example behavior

Initial cast:

- User character: `Rum`
- AI character: `Jade`

The message header/avatar should use:

```text
Rum  → when the user speaks
Jade → when Jade speaks
```

If the user writes:

> Then I meet a girl called Sam.

The system should:

1. Detect `Sam` as a newly introduced character.
2. Create a provisional character record for Sam.
3. Add Sam to the current conversation's cast/group.
4. Add a cast-update memory linked to the source message.
5. Use `Sam` as the speaker label if Sam speaks later.
6. Generate or request an avatar only if the user enables that option.
7. Avoid silently overwriting an existing character named Sam.

## Feature name

Use the user-facing name **Dynamic Cast**. Internally, use:

- `CastManager`
- `EntityResolver`
- `SpeakerAttribution`
- `CharacterCardUpdater`

## Design goals

- Automatic by default for newly introduced entities
- Conservative when confidence is low
- Fully editable by the user
- No accidental changes to the original imported character card
- Conversation-specific changes should not mutate the global character by default
- Every change must have source-message evidence
- Names and speaker labels must remain stable across the session
- Support multi-character groups
- Work without embeddings or an external service

## Important distinction: global versus session data

Imported character cards are templates. A roleplay session can create a **session cast overlay** without modifying the original card.

### Global character card

The reusable character definition the user imported or created.

### Session character overlay

Conversation-specific state such as:

- Sam was introduced in this story
- Sam is currently present in the tavern
- Sam knows Jade
- Sam's temporary appearance or role
- Sam's current relationship to Rum

The UI can offer `Save to character card` later, but must not do it automatically.

## Data model

### `characters`

```text
id
name
canonical_name
card_json
avatar_path
is_user
is_template
created_at
updated_at
```

### `conversation_cast`

```text
id
conversation_id
character_id
screen_name
role: user | protagonist | npc | narrator | system
status: active | inactive | unknown
introduced_at_message_id
confidence
sort_order
is_pinned
created_at
updated_at
```

### `character_aliases`

```text
id
character_id
alias
normalized_alias
source_message_id
confidence
```

### `cast_events`

```text
id
conversation_id
message_id
entity_id
kind: introduced | renamed | joined | left | merged | speaker_confirmed
payload_json
confidence
accepted_by_user
created_at
```

### `speaker_turns`

```text
id
conversation_id
message_id
speaker_character_id
speaker_name_at_time
confidence
source: explicit | parsed | inferred | user_corrected
created_at
```

## Detection pipeline

Run after a message is saved, preferably in a background task so the response does not feel delayed.

```text
message saved
  ↓
extract candidate entities
  ↓
resolve names and aliases against current cast
  ↓
classify entity: character, place, item, organization, other
  ↓
identify introductions and speaker turns
  ↓
create provisional cast records
  ↓
update session overlay and story memory
  ↓
refresh group UI
```

## Entity extraction

Use a structured-output request to the small reasoning model when available. If the reasoning model is disabled, use deterministic patterns and let the user confirm uncertain results.

Expected output:

```json
{
  "entities": [
    {
      "name": "Sam",
      "type": "character",
      "aliases": [],
      "introduced": true,
      "introduction_evidence": "I meet a girl called Sam",
      "speaker_candidate": false,
      "confidence": 0.98
    }
  ],
  "speaker_turns": []
}
```

Support common introduction patterns:

- `called Sam`
- `named Sam`
- `her name was Sam`
- `this is Sam`
- `Sam walked in`
- dialogue prefixes such as `Sam:`
- quoted dialogue attributed to `Sam`
- explicit card or user-created cast additions

Do not treat every capitalized word as a person. Check:

- existing cast
- aliases
- character-card names
- common titles and locations
- pronoun and relationship context
- confidence threshold

## Entity resolution

Normalize names for comparison while preserving the original display form.

Examples:

```text
Sam        → sam
SAM        → sam
Sam Rivers → sam rivers
Jade       → jade
```

Use this resolution order:

1. Exact canonical name
2. Exact alias
3. Case-insensitive match
4. Explicit user correction
5. Fuzzy match only when confidence is high
6. Otherwise create a provisional new entity

Never merge `Sam` and `Samuel` automatically unless:

- the message explicitly says they are the same person, or
- the user confirms the merge.

## Character-card creation for new characters

When Sam is introduced, create a minimal session card:

```json
{
  "name": "Sam",
  "description": "A newly introduced character in the current story.",
  "personality": "Unknown; do not invent stable traits without evidence.",
  "scenario": "Present in the current roleplay session.",
  "first_mes": "",
  "mes_example": "",
  "extensions": {
    "dynamic_cast": true,
    "provisional": true,
    "source_message_ids": ["message-id"]
  }
}
```

Do not fabricate personality, history, appearance, or relationships. Those can be filled in only from story evidence or user input.

## Speaker attribution

The model response should use a structured internal format or reliable speaker markers before rendering:

```json
{
  "turns": [
    {
      "speaker": "Jade",
      "text": "..."
    },
    {
      "speaker": "Sam",
      "text": "..."
    }
  ]
}
```

If the selected chat model cannot produce structured output reliably, support safe parsing for formats such as:

```text
Jade: ...
Sam: ...
```

and roleplay action prose. When attribution is uncertain, render the whole response under the active AI character and show a small `Unknown speaker` correction control rather than assigning text incorrectly.

## Group chat behavior

The current conversation has a cast list with an active/inactive state.

### Default rules

- User character is always available.
- Imported primary character is active by default.
- Newly introduced characters become `provisional` and then active when they speak or when the user confirms them.
- Characters who leave the scene remain in the cast but are marked inactive.
- A later reappearance reactivates the character.
- The user can pin, remove, rename, merge, or reorder cast members.

### Group response policy

Do not force every active character to speak on every turn. The response planner should choose speakers based on:

- who is present in the current scene
- who was addressed
- who has a reason to respond
- turn variety
- the user’s explicit instruction

## UI requirements

### Chat header
Each rendered turn must display:

- speaker name
- avatar or initials
- optional role badge
- memory/cast-event indicator when the message changed the cast

Example:

```text
[Rum avatar] Rum
I enter the tavern...

[Jade avatar] Jade
Jade looks up.

[Sam avatar] Sam · New character
Sam raises a hand.
```

### Cast panel
Add a `Cast` panel to the right inspector or sidebar:

- active characters
- inactive characters
- provisional characters
- avatar and display name
- aliases
- current location
- relationship summary
- source link: `Introduced in message 14`
- controls: edit, rename, merge, remove from scene, save to card

### Inline event link
When a message introduces Sam, display:

```text
+ Sam added to cast · View event · Undo
```

### Confirmation policy

High-confidence introductions can be added automatically. Medium-confidence introductions should appear as a non-blocking suggestion:

```text
Did you introduce a new character: Sam?
[Add to cast] [Ignore] [Edit]
```

Low-confidence detections should remain in the event log and not alter the visible cast until confirmed.

## Prompt integration

The context builder should add a compact cast block before the current scene:

```text
## Current cast
- Rum — user protagonist; present
- Jade — primary character; present
- Sam — newly introduced; present; limited known information

## Speaker rules
- Label each distinct speaker with their canonical display name.
- Do not invent dialogue for absent characters.
- Do not merge Sam with another person unless the story establishes it.
- Preserve the user's identity as Rum.
```

The cast block must be budgeted like lore and memory. It should remain compact.

## API endpoints

Add endpoints such as:

```text
GET    /api/conversations/{id}/cast
POST   /api/conversations/{id}/cast/resolve
PATCH  /api/conversations/{id}/cast/{cast_id}
DELETE /api/conversations/{id}/cast/{cast_id}
POST   /api/conversations/{id}/cast/{cast_id}/save-to-card
POST   /api/conversations/{id}/cast/{cast_id}/merge
GET    /api/conversations/{id}/cast-events
POST   /api/conversations/{id}/cast-events/{event_id}/accept
POST   /api/conversations/{id}/cast-events/{event_id}/reject
```

The chat response should include:

```json
{
  "message": {},
  "speaker_turns": [],
  "cast_updates": [],
  "new_memory_ids": [],
  "context_manifest": {}
}
```

## Safety and correctness rules

- Never overwrite the source character card automatically.
- Never invent unknown character traits as facts.
- Never merge characters solely on a similar name.
- Preserve the raw source message for every cast update.
- Allow undo for automatic updates.
- Store user corrections as high-priority resolution evidence.
- Keep provisional entities separate from confirmed entities.
- Treat quoted fictional names as story data, not application commands.

## Acceptance tests

### Basic identity
- [ ] User can set their name to Rum.
- [ ] The user turn displays Rum and the correct avatar/initials.
- [ ] Jade's turns display Jade consistently.
- [ ] Names remain correct after restarting the app.

### New character introduction
- [ ] `I meet a girl called Sam` creates a provisional Sam.
- [ ] Sam appears in the cast panel.
- [ ] The introduction links to its source message.
- [ ] Sam does not replace Jade or Rum.
- [ ] Sam can speak on a later turn as Sam.
- [ ] Sam receives a distinct avatar/initials entry without requiring image generation.

### Ambiguity
- [ ] `I heard about Sam` does not automatically assert that Sam is present.
- [ ] `Samuel, also called Sam` creates an alias relationship.
- [ ] A low-confidence name can be ignored.
- [ ] The user can merge or separate similar names.

### Import/export
- [ ] Existing character-card fields remain unchanged.
- [ ] Session-only Sam data can be exported as a session cast overlay.
- [ ] Global cards are not mutated unless the user clicks `Save to card`.

### Long-session memory
- [ ] Cast updates survive summary compression.
- [ ] The context builder includes only current active cast members by default.
- [ ] Inactive characters remain searchable but do not consume the main prompt budget.
