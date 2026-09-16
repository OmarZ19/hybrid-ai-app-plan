# Structured Roleplay Implementation Checklist

## Phase 1: Backend schema

- [ ] Add `roleplay_turns` table.
- [ ] Add `turn_type` enum/check constraint.
- [ ] Add foreign keys to conversations, messages, characters, and parent turns.
- [ ] Add indexes on `(conversation_id, sequence)` and `(message_id)`.
- [ ] Add migration and rollback.

## Phase 2: Response contract

- [ ] Add Pydantic models: `RoleplayTurn`, `StructuredRoleplayResponse`, `SpeakerPlan`.
- [ ] Add model-provider capability flag for structured JSON output.
- [ ] Add JSON schema to providers that support it.
- [ ] Add fallback prompt for providers without schema support.

## Phase 3: Normalization

- [ ] Implement `TurnNormalizer`.
- [ ] Implement canonical name and alias resolution.
- [ ] Validate speaker IDs against the conversation cast.
- [ ] Repair or reject malformed output.
- [ ] Preserve raw fragments for diagnostics.
- [ ] Add deterministic sequence numbers.

## Phase 4: Group planner

- [ ] Implement `GroupResponsePlanner`.
- [ ] Respect maximum-speaker setting.
- [ ] Track recent speaker frequency to avoid repetitive turns.
- [ ] Do not select inactive or absent characters.
- [ ] Support user-directed speaker overrides.
- [ ] Return planner explanation in the context manifest.

## Phase 5: Frontend

- [ ] Implement `UserTurn`.
- [ ] Implement `CharacterTurn`.
- [ ] Implement `NarratorTurn`.
- [ ] Implement `RoleplayTurnGroup`.
- [ ] Add speaker avatars and stable names.
- [ ] Add turn-level memory and cast-event links.
- [ ] Add edit/regenerate/branch controls.
- [ ] Add a compact narrator style that reads like prose.

## Phase 6: Memory and export

- [ ] Extract memories from individual turns.
- [ ] Link world-state changes to narrator turns.
- [ ] Link relationship facts to character turns.
- [ ] Preserve turn types in app-native exports.
- [ ] Add optional flattened compatibility export.

## Tests

### Unit tests

- [ ] Parse valid structured response.
- [ ] Reject invalid turn type.
- [ ] Resolve aliases correctly.
- [ ] Reject unknown character speaker.
- [ ] Preserve narrator text separately.
- [ ] Remove empty turns.
- [ ] Repair a missing sequence.

### Integration tests

- [ ] Rum user turn renders as Rum.
- [ ] Jade character turn renders as Jade.
- [ ] Narrator prose renders separately below Jade.
- [ ] Sam is added to the cast after introduction.
- [ ] Sam and Jade respond in one response with separate turns.
- [ ] Memories link to exact turns.
- [ ] Conversation reload preserves all turn types.
- [ ] Export and re-import preserve the same sequence.

### Regression example

Input:

```text
Rum: Then I meet a girl called Sam.
```

Expected normalized result after generation:

```json
[
  {
    "type": "character",
    "speaker_name": "Sam",
    "text": "Hello, Rum."
  },
  {
    "type": "narrator",
    "speaker_name": "Narrator",
    "text": "Sam steps into the lantern light."
  },
  {
    "type": "character",
    "speaker_name": "Jade",
    "text": "You know her?"
  },
  {
    "type": "narrator",
    "speaker_name": "Narrator",
    "text": "Jade's gaze moves from Rum to Sam."
  }
]
```

## Double-check before declaring complete

Run all of the following:

1. Run backend unit and integration tests.
2. Start the app and manually test Rum, Jade, and Sam.
3. Introduce Sam in the middle of a long conversation.
4. Confirm Sam appears in the cast panel and speaks with a distinct identity.
5. Confirm Jade can respond in the same assistant generation.
6. Confirm each prose block is visibly separate from dialogue.
7. Reload the conversation and verify the exact sequence remains.
8. Export and re-import the conversation.
9. Inspect the context manifest and verify no unknown speaker was injected.
10. Run the long-session memory test after structured turns are enabled.
