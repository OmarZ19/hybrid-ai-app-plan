# Better SkillOpt Integration

## Decision

SkillOpt is useful, but it should **not** run inside the normal chat request path and should not be treated as a general-purpose memory system.

Use SkillOpt as an **offline, opt-in skill compiler** that improves reusable roleplay procedures. Keep memory and SkillOpt separate:

- Memory stores what happened.
- Lorebooks store authored world knowledge.
- Character cards store identity and voice.
- SkillOpt improves how the agent performs repeatable tasks.

This separation prevents the optimizer from rewriting personal memories or character canon.

## Recommended role for SkillOpt

Optimize small, reusable roleplay skills such as:

- scene continuity checking
- concise memory extraction
- relationship-state updates
- lorebook selection
- dialogue style preservation
- conflict resolution between old and new facts
- summary compression
- long-session recap generation

Do not optimize the character card itself by default. Character identity should remain user-controlled.

## Offline optimization flow

```text
Conversation transcript + user feedback + evaluation fixtures
                         ↓
                  SkillOpt adapter
                         ↓
              candidate skill edit(s)
                         ↓
             local roleplay evaluation suite
                         ↓
             validation gate / score comparison
                         ↓
        accept best skill or reject candidate edit
                         ↓
       versioned skill artifact in local skill library
```

The optimizer runs only when:

- the user presses `Improve skill`
- a scheduled offline job is enabled
- a developer runs an evaluation command

It never silently changes behavior during a live scene.

## Skill artifact layout

```text
skills/
  roleplay/
    continuity_checker/
      current_skill.md
      best_skill.md
      history.jsonl
      fixtures.jsonl
      scores.jsonl
    memory_extractor/
      current_skill.md
      best_skill.md
      history.jsonl
      fixtures.jsonl
      scores.jsonl
    relationship_tracker/
      current_skill.md
      best_skill.md
      history.jsonl
      fixtures.jsonl
      scores.jsonl
```

## Skill manifest

Each skill should have a manifest:

```json
{
  "id": "roleplay.memory_extractor",
  "version": 1,
  "purpose": "Extract durable roleplay memories with evidence links",
  "input_contract": "recent conversation segment plus current state",
  "output_contract": "validated JSON memory candidates",
  "enabled": true,
  "optimizer_enabled": false,
  "model": "small-reasoning-model",
  "max_skill_tokens": 1200,
  "created_at": "...",
  "updated_at": "..."
}
```

## Evaluation gates

A candidate skill must beat the current skill on a held-out local fixture set.

Use a score with weighted components:

```text
score =
  0.30 * fact_precision
+ 0.25 * fact_recall
+ 0.20 * relationship_consistency
+ 0.15 * summary_compression
+ 0.10 * output_format_validity
```

For continuity-checking skills, increase relationship and state consistency. For compression skills, increase fact recall and compression quality.

Never accept a candidate solely because it produces a longer answer.

## Adapter boundary

Create a narrow adapter instead of importing SkillOpt into every backend module:

```python
class SkillOptimizer:
    def improve(
        self,
        skill_id: str,
        fixtures_path: str,
        *,
        dry_run: bool = False,
    ) -> dict:
        """Return candidate, scores, and acceptance decision."""
```

The adapter should:

- invoke SkillOpt through a subprocess or stable public API
- use a temporary working directory
- never overwrite `best_skill.md` before validation
- save candidate, scores, and logs
- support cancellation
- enforce a time and token budget
- return a clear error if SkillOpt is unavailable

## CLI commands

Add commands such as:

```text
rpapp skill list
rpapp skill test roleplay.memory_extractor
rpapp skill improve roleplay.memory_extractor --dry-run
rpapp skill promote roleplay.memory_extractor
rpapp skill rollback roleplay.memory_extractor --version 2
```

The UI can call the same service and should show:

- current version
- best version
- last score
- validation score
- changed lines
- accept/reject reason
- rollback button

## Runtime behavior

At runtime, use only the selected `best_skill.md` artifact. Do not run the optimizer during a response.

A runtime request should look like:

```text
system instructions
+ character card
+ selected roleplay skill
+ world state
+ selected lorebook entries
+ retrieved memories
+ compressed history
+ recent turns
+ current user message
```

The skill should be short. Keep it within a configured token cap so skill instructions do not consume the context needed for the scene.

## Memory safety rules

SkillOpt must never:

- edit raw transcripts
- delete memories
- rewrite character cards automatically
- change user safety settings
- change export data silently
- promote a candidate without validation
- add hidden instructions that the user cannot inspect

## Implementation phases

### Phase A: Native skill registry
- Add skill manifest schema.
- Load Markdown skills from local folders.
- Add enable/disable controls.
- Include selected skill in context manifest.

### Phase B: Local evaluation
- Add fixtures for memory extraction, continuity, and summary compression.
- Add deterministic checks for JSON validity, evidence IDs, and required fields.
- Add model-based scoring only as an optional layer.

### Phase C: SkillOpt adapter
- Detect whether SkillOpt is installed.
- Invoke it only in offline mode.
- Materialize candidate edits in a temporary directory.
- Run validation gate.
- Persist version history.

### Phase D: UI
Add a Skills panel with:

- enabled skills
- skill purpose
- current/best version
- score history
- improve button
- dry-run preview
- rollback

## Why this is better

This integration uses SkillOpt where it is strongest: evolving reusable procedural guidance against scored examples. It avoids using SkillOpt as a live memory database, avoids latency in roleplay responses, preserves user control, and provides a measurable path to improvement.
