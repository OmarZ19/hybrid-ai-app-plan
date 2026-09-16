# Optional Integration Matrix

| Integration | Default | Benefit | Cost / Risk | Recommended use |
|---|---:|---|---|---|
| Native SQLite memory | Yes | Offline, simple, inspectable | Requires our own retrieval code | Core memory system |
| Character-card importer/exporter | Yes | Compatibility with roleplay ecosystem | Format variants | Core roleplay feature |
| Lorebook engine | Yes | Efficient world context injection | Trigger tuning | Core roleplay feature |
| Structured summaries | Yes | Compresses long sessions | Summary errors | Core memory feature |
| Hybrid lexical retrieval | Yes | Fast exact recall | Less semantic flexibility | Default retrieval |
| Embeddings | No | Better semantic recall | Extra model/storage | Optional enhancement |
| Qdrant/Chroma | No | External vector search | Extra service | Power users |
| Letta adapter | No | Hierarchical agent memory | More infrastructure | Optional provider |
| Mem0 adapter | No | Memory add/update workflow | External dependency/API changes | Optional provider |
| Graph memory | No | Relationships and temporal queries | Complexity | Experimental |
| SkillOpt optimizer | No | Improves reusable skills | Offline evaluation cost | Opt-in skill training |
| Emotion state | No | More consistent character affect | Can become distracting | Optional RP enhancement |
| Multi-agent system | No | Specialized roleplay workers | Complexity and latency | Future feature |

## Default-first principle

A fresh install should work with only:

- SQLite
- one model runtime
- one character card
- built-in memory extraction
- built-in lorebook activation
- structured summarization

Everything else must degrade gracefully or remain disabled until explicitly enabled.
