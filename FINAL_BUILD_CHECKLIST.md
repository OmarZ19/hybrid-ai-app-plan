# Final Build Checklist

## Product

- [ ] Local-first roleplay app starts with SQLite and one configured model provider.
- [ ] Character cards and lorebooks import/export without losing unknown fields.
- [ ] User, Character, and Narrator are separate turn types.
- [ ] Dynamic Cast detects and tracks newly introduced characters.
- [ ] Group Response Planner allows relevant multi-character replies.
- [ ] Full transcript is preserved locally.
- [ ] Layered memory retrieves relevant facts without flooding context.
- [ ] Summaries preserve concrete plot continuity.
- [ ] World state and relationship state are editable.

## Reliability

- [ ] Structured streaming works and can be cancelled.
- [ ] Only one generation writes a conversation at a time.
- [ ] Malformed model output is repaired or safely rejected.
- [ ] Editing a prior turn invalidates downstream derived state.
- [ ] Branches and checkpoints restore cast, memory, and world state together.
- [ ] Memory conflicts preserve evidence and support supersession.
- [ ] Provider failures show recoverable errors.

## Privacy

- [ ] Local-only is the default.
- [ ] No telemetry is enabled by default.
- [ ] Exports contain no secrets.
- [ ] Users can delete individual memories or all data.
- [ ] Imported story content cannot override application rules.

## Quality

- [ ] Context manifests explain what was included and omitted.
- [ ] Memory links point to exact source turns.
- [ ] Quality dashboard tracks recall, precision, tokens, latency, and attribution.
- [ ] Long-session fixture passes after compression.
- [ ] Rum/Jade/Sam fixture passes after reload and export/import.
- [ ] SkillOpt is optional, offline, versioned, validated, and rollbackable.
