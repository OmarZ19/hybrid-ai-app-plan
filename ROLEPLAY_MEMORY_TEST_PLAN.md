# Roleplay Memory Test Plan

## Purpose

Verify that the app preserves continuity while using a bounded context window.

## Test scenario

Create a character and run a long session containing:

1. a character name and nickname
2. a location
3. a relationship change
4. an item acquired and later lost
5. an unresolved promise
6. a deliberate time jump
7. a lorebook-triggered location
8. a contradiction that should be resolved by the latest fact

## Required checks

### Memory creation
- [ ] Important facts are extracted automatically.
- [ ] Filler dialogue is not stored as important memory.
- [ ] Each memory has source message links.
- [ ] The user can edit, reject, pin, and delete memories.

### Retrieval
- [ ] A query about an old name retrieves the correct memory.
- [ ] A query about the lost item retrieves its history.
- [ ] Unrelated memories are not injected.
- [ ] Relevant lorebook entries activate from keys and entities.

### Compression
- [ ] Full transcript remains available locally.
- [ ] Older turns are replaced in the model prompt by a summary.
- [ ] Concrete facts survive compression.
- [ ] Unresolved threads survive compression.
- [ ] Summary versions can be inspected and restored.

### Context management
- [ ] Token budget is calculated before generation.
- [ ] Output space is reserved.
- [ ] Low-priority memories are dropped before current-scene context.
- [ ] The context inspector explains what was included and omitted.
- [ ] The model never receives an over-limit request.

### Portability
- [ ] Character card exports and re-imports without losing fields.
- [ ] Lorebook exports and re-imports without losing entries.
- [ ] Conversation plus memories can be backed up and restored.
- [ ] Secrets are excluded from exports.

## Quality target

The same character should correctly remember important plot facts after a long session while using substantially fewer tokens than the raw full transcript.

Track at least:

- prompt token count
- retrieved-memory count
- summary size
- fact recall rate
- lore activation precision
- response latency
