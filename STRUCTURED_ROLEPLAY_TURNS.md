# Structured Roleplay Turn System

## Objective

Separate roleplay output into explicit `User`, `Character`, and `Narrator` turns so dialogue and prose are never jumbled together. Each character's speech is rendered as its own message, followed immediately by the narration that describes the action or consequence of that character's turn.

## Required output model

The backend must return structured turns instead of one untyped response string.

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
      "text": "Jade lowers the lantern, studying Rum's expression.",
      "sequence": 2,
      "follows_turn": 1
    },
    {
      "type": "character",
      "speaker_id": "sam",
      "speaker_name": "Sam",
      "text": "You know him?",
      "sequence": 3
    },
    {
      "type": "narrator",
      "speaker_id": "narrator",
      "speaker_name": "Narrator",
      "text": "Sam takes one cautious step into the light.",
      "sequence": 4,
      "follows_turn": 3
    }
  ]
}
```

## Turn types

### User

Represents the user's in-story action or dialogue. The UI displays the configured user identity, such as `Rum`.

```json
{
  "type": "user",
  "speaker_id": "rum",
  "speaker_name": "Rum",
  "text": "I enter the tavern.",
  "sequence": 1
}
```

### Character

Represents in-character dialogue from a member of the active cast. A character turn must always contain a validated `speaker_id` that belongs to the conversation cast.

### Narrator

Represents prose, scene description, environmental change, action, consequence, or transitions. Narrator text must never be placed inside a character dialogue bubble.

Narrator mode should be configured separately from character voice:

- third-person narrator
- second-person narrator
- cinematic narrator
- user-authored narrator

## Rendering rules

The frontend must render each structured turn separately:

```text
[Rum avatar] Rum
I enter the tavern.

[Jade avatar] Jade
You came back.

[Narrator icon] Narrator
Jade lowers the lantern, studying Rum's expression.

[Sam avatar] Sam
You know him?

[Narrator icon] Narrator
Sam takes one cautious step into the light.
```

Required behavior:

- Character dialogue uses the character's name and avatar.
- User dialogue/action uses the configured user name and avatar.
- Narration uses a visually distinct prose block.
- Narration is shown directly beneath the character turn it follows.
- Multiple speakers remain in sequence order.
- Narration never gets merged into a preceding or following dialogue block.
- Empty turns are discarded.
- Markdown formatting is preserved within each turn but cannot change its turn type.

## Generation contract

The chat model should be asked for JSON using a strict schema. The model must not decide arbitrary speaker IDs. It may propose names, but the backend resolves them against the active cast.

```json
{
  "type": "object",
  "required": ["turns"],
  "properties": {
    "turns": {
      "type": "array",
      "minItems": 1,
      "maxItems": 12,
      "items": {
        "type": "object",
        "required": ["type", "speaker_name", "text"],
        "properties": {
          "type": {
            "type": "string",
            "enum": ["character", "narrator"]
          },
          "speaker_name": {"type": "string"},
          "text": {"type": "string"}
        }
      }
    }
  }
}
```

The prompt must include these rules:

```text
Return only valid JSON matching the supplied schema.
Separate every spoken line from narration.
Use a character turn for dialogue only.
Use a narrator turn for actions, setting, body language, transitions, and consequences.
After a character speaks, put the prose describing that character's action immediately after it.
Do not put dialogue inside a narrator turn.
Do not invent a speaker who is not in the active cast.
Do not make every active character speak automatically; only include relevant speakers.
```

## Backend normalization and validation

Never trust model output directly. Add a `TurnNormalizer` service with this flow:

```text
raw model response
  → parse JSON
  → validate schema
  → resolve speaker names against cast
  → classify unknown/ambiguous speakers
  → discard empty turns
  → merge adjacent narrator turns only when they belong together
  → ensure narrator follows the relevant character when possible
  → assign stable IDs and sequence numbers
  → persist normalized turns
  → return to UI
```

### Speaker resolution

1. Match canonical character ID.
2. Match known alias.
3. Match case-insensitive display name.
4. Ask the cast resolver for a high-confidence match.
5. If unresolved, convert the turn to `narrator` only when it contains no dialogue; otherwise mark the response for repair.

The backend must reject or repair these cases:

- character name not in active cast
- `Narrator` speaking dialogue
- character turn containing unmarked prose plus dialogue
- missing `type`
- empty text
- invalid JSON
- duplicate or unstable speaker IDs

## Repair strategy

If parsing fails:

1. Attempt one local JSON repair.
2. If still invalid, request a concise structured retry from the same model.
3. If retry fails, preserve the raw response in diagnostics and render it as an assistant error state rather than silently mixing prose and dialogue.

Do not use a fragile regex-only parser as the primary path.

## Group response planner

Add a `GroupResponsePlanner` before generation. It selects relevant speakers and controls ordering.

Inputs:

- active cast
- current scene state
- addressed character
- latest user turn
- recent speaker history
- character goals and relationships
- maximum speakers setting

Outputs:

```json
{
  "speakers": ["jade", "sam"],
  "include_narrator": true,
  "reason": "Jade was addressed; Sam is present and has a relevant reaction.",
  "max_speakers": 2
}
```

Defaults:

- multiple speakers: enabled
- maximum character speakers: 2
- narrator: enabled
- minimum reason to speak: required
- active characters do not all speak by default

A user can override the planner with commands such as:

- `Jade responds`
- `Have Sam and Jade react`
- `Narrator only`
- `Everyone stays silent except Sam`

## Data model additions

Add a normalized `roleplay_turns` table:

```text
id
message_id
conversation_id
sequence
turn_type: user | character | narrator
speaker_character_id nullable
speaker_name
text
follows_turn_id nullable
raw_fragment nullable
created_at
```

Keep the original assistant response in `messages` for auditability, but use `roleplay_turns` as the source for rendering, memory extraction, export, and replay.

## Memory integration

Memory extraction must understand all three types:

- User turns reveal user actions, choices, and facts.
- Character turns reveal dialogue, knowledge, promises, and relationships.
- Narrator turns reveal world-state changes, locations, item changes, injuries, time, and consequences.

Every extracted memory should link to the exact `roleplay_turn_id`, not only the whole assistant message.

Narrator turns should be prioritized for:

- current location
- time progression
- physical actions
- inventory changes
- world events
- consequences

Character turns should be prioritized for:

- spoken commitments
- revealed knowledge
- relationship changes
- emotional declarations
- character-specific facts

## UI requirements

### Turn components
Create separate components:

- `UserTurn`
- `CharacterTurn`
- `NarratorTurn`
- `RoleplayTurnGroup`

`RoleplayTurnGroup` visually associates a character turn with the following narrator prose while keeping them as separate accessible elements.

### Controls
Every turn should support:

- copy
- edit
- regenerate from here
- branch from here
- link to memories
- link to cast events
- show raw structured data in developer mode

### Styling

- User turn: user identity color and avatar
- Character turn: character avatar, name, and optional emotion badge
- Narrator turn: centered or inset prose with distinct typography, muted color, and no dialogue bubble
- Avoid mixing narrator text into character bubble styling

## Export rules

Conversation export must preserve turn types:

```json
{
  "format": "hybrid-rp-v1",
  "conversation": {},
  "turns": [
    {
      "type": "character",
      "speaker": "Jade",
      "text": "..."
    },
    {
      "type": "narrator",
      "speaker": "Narrator",
      "text": "..."
    }
  ]
}
```

For character-card or SillyTavern-compatible exports, provide an explicit choice:

- preserve structured turns in the app-native export
- flatten turns for compatibility with clients that only support text messages

Never flatten the canonical stored version.

## Acceptance criteria

- [ ] User, Character, and Narrator are separate turn types in the database and API.
- [ ] Dialogue is never rendered as narration.
- [ ] Narration is never rendered inside a character's dialogue bubble.
- [ ] Each character's prose follows that character's dialogue when generated together.
- [ ] Sam and Jade can both respond in the same assistant message with correct identities.
- [ ] Active cast validation prevents unknown speakers.
- [ ] Memory links point to individual normalized turns.
- [ ] Export/import preserves turn types.
- [ ] Malformed model output cannot corrupt the chat transcript.
- [ ] The app remains usable when structured generation is unavailable.
