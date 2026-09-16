# Suggested Runtime Implementation

## Service boundaries

```text
backend/
  roleplay/
    cast_manager.py
    entity_resolver.py
    speaker_attribution.py
    character_overlay.py
    cast_events.py
    cast_prompts.py
```

## Suggested service interfaces

```python
class CastManager:
    async def inspect_message(
        self,
        conversation_id: str,
        message_id: str,
        text: str,
    ) -> CastInspection:
        """Extract entities, speaker candidates, and proposed cast events."""

    async def apply_event(
        self,
        event_id: str,
        decision: Literal["accept", "reject", "edit"],
    ) -> CastEventResult:
        """Apply or reject an event with an undoable audit record."""

    async def get_prompt_cast(self, conversation_id: str) -> PromptCast:
        """Return a compact active-cast block for the context builder."""
```

## Do not block chat on every operation

Recommended sequence:

1. Save the user message.
2. Run fast deterministic name detection.
3. Build the prompt using the currently known cast.
4. Generate the response.
5. Attribute response speakers.
6. Run deeper entity extraction in a background task.
7. Apply high-confidence events automatically.
8. Notify the UI about cast changes.

This means a newly introduced Sam may appear in the cast panel immediately after the user message, while deeper relationship details can be added shortly afterward.

## Avatar behavior

Do not require image generation.

Default:

- derive initials from the canonical name
- use a deterministic color based on character ID
- allow an imported avatar from a character card

Optional:

- generate an avatar through a configured local image model
- choose from preset avatars

## Suggested event payload

```json
{
  "event_type": "introduced",
  "conversation_id": "conv-1",
  "message_id": "msg-14",
  "entity": {
    "name": "Sam",
    "type": "character",
    "aliases": []
  },
  "evidence": "I meet a girl called Sam",
  "confidence": 0.98,
  "auto_apply": true,
  "reversible": true
}
```

## Prompt parser fallback

Models that support structured output should receive a JSON schema. For other models:

- ask for a response using `SPEAKER:` lines
- parse only known cast names
- preserve unrecognized text as the primary character response
- never treat arbitrary model prose as a database command

## Implementation order

1. Add database tables and migrations.
2. Add current user identity and primary-character identity.
3. Add cast panel and name/avatar rendering.
4. Add deterministic introduction pattern detection.
5. Add confirmation and undo.
6. Add structured model extraction.
7. Add response speaker attribution.
8. Add session overlay export.
9. Add optional save-to-global-card.
10. Add long-session continuity tests.
