# 05 - Memory

## Overview

OpenClaw supports two complementary memory systems:

1. **Workspace MEMORY.md** -- a curated markdown file you edit manually. Good for stable facts, preferences, and project context.
2. **LanceDB vector memory** (via the `memory-lancedb-pro` plugin) -- a semantic vector store the agent reads and writes at runtime. Good for accumulating knowledge across many conversations.

Both can be used simultaneously. MEMORY.md provides reliable baseline context; LanceDB provides scalable semantic recall.

## LanceDB Plugin Setup

### 1. Enable the plugin in openclaw.json

```jsonc
{
  "plugins": {
    "memory-lancedb-pro": {
      "enabled": true,
      "jinaApiKey": "${JINA_API_KEY}",
      "dbPath": "./data/memory-lancedb"
    }
  }
}
```

### 2. Set the Jina API key

memory-lancedb-pro uses Jina AI embeddings to encode text into vectors.

```bash
# In your .env file
JINA_API_KEY=jina_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

Get a key at https://jina.ai/ (free tier available).

### 3. Verify it works

```bash
openclaw chat
> Remember that my preferred language is Python.
# Agent should call memory_store

> What is my preferred language?
# Agent should call memory_recall and find "Python"
```

## Jina Embeddings Configuration

The default embedding model is `jina-embeddings-v3`. Configuration options in openclaw.json:

```jsonc
{
  "plugins": {
    "memory-lancedb-pro": {
      "enabled": true,
      "jinaApiKey": "${JINA_API_KEY}",
      "dbPath": "./data/memory-lancedb",
      "embeddingModel": "jina-embeddings-v3",
      "embeddingDimensions": 1024
    }
  }
}
```

Higher dimensions improve recall accuracy but increase storage and latency. 1024 is a good default.

## Core Operations

### memory_store

Saves a piece of information to the vector store.

```
memory_store({
  "content": "User prefers Python over JavaScript for backend work",
  "category": "preference",
  "importance": 0.8,
  "tags": ["language", "backend"]
})
```

**Parameters:**
| Field | Required | Description |
|-------|----------|-------------|
| content | Yes | The text to store. Be specific and self-contained. |
| category | No | One of: `fact`, `decision`, `preference`, `event`, `note` |
| importance | No | Float 0.0-1.0. Higher = more resistant to decay. Default: 0.5 |
| tags | No | Array of strings for filtering. |

**When to store:**
- User corrections ("Actually, I use PostgreSQL, not MySQL")
- Key decisions ("We decided to use REST instead of GraphQL")
- Stated preferences ("I prefer concise answers")
- Important project facts ("The deploy target is AWS ECS in us-east-1")

**When NOT to store:**
- Transient conversation details ("Let me think about that")
- Information already in MEMORY.md
- Trivial facts that will not be useful later

### memory_recall

Retrieves relevant memories via semantic search.

```
memory_recall({
  "query": "what database does the user prefer",
  "limit": 5
})
```

**Parameters:**
| Field | Required | Description |
|-------|----------|-------------|
| query | Yes | Natural language query. Semantic, not keyword-based. |
| limit | No | Max results to return. Default: 10 |
| category | No | Filter by category. |
| tags | No | Filter by tags. |

**When to recall:**
- Start of a new conversation (broad recall: "user preferences and project context")
- Before making assumptions ("what language does the user prefer")
- When the user references something from a past conversation

## Cross-Session Semantic Recall

The key advantage of vector memory over plain text: the agent can find relevant information even when the wording is different.

Example:
```
# Session 1
User: "We're using Postgres 16 on RDS."
Agent: [stores: "Production database is PostgreSQL 16 running on AWS RDS"]

# Session 2 (days later)
User: "What's our DB setup?"
Agent: [recalls with query "database setup"] -> finds the PostgreSQL fact
```

This works because the embedding model encodes meaning, not exact words. "DB setup" is semantically close to "Production database is PostgreSQL 16."

## Memory Decay and Importance Scoring

memory-lancedb-pro implements a decay model to prevent unbounded memory growth:

- **Importance** (0.0-1.0): Set at store time. Higher importance decays slower.
- **Recency**: Recently stored or recalled memories rank higher.
- **Access count**: Memories that are recalled frequently decay slower.
- **Effective score**: Combines importance, recency, and access count. Memories below a threshold are pruned during periodic cleanup.

**Practical guidance:**
| Importance | Use for |
|------------|---------|
| 0.9-1.0 | Critical corrections, security-relevant facts |
| 0.7-0.8 | User preferences, key project decisions |
| 0.5-0.6 | General facts, context notes |
| 0.3-0.4 | Observations, minor details |

Set importance deliberately. The default (0.5) is fine for general notes, but corrections and preferences should be 0.8+.

## Workspace MEMORY.md vs LanceDB

| Aspect | MEMORY.md | LanceDB |
|--------|-----------|---------|
| Who writes it | You (manually) | The agent (via memory_store) |
| Format | Markdown | Vector embeddings + metadata |
| Search | Exact text (model reads the whole file) | Semantic similarity |
| Scale | ~2-4 KB practical limit | Thousands of entries |
| Reliability | Fully deterministic | Depends on embedding quality |
| Persistence | File on disk | Database on disk |
| Use case | Stable context, project facts | Accumulated knowledge, corrections |

**Best practice:** Use both.
- Put stable, critical information in MEMORY.md (project overview, key contacts, deployment info).
- Let the agent use LanceDB for everything it learns during conversations.
- Periodically review LanceDB contents and promote important entries to MEMORY.md.

## Troubleshooting

**Memory recall returns no results:**
- Verify the plugin is enabled (`"enabled": true`)
- Check that the Jina API key is valid
- Confirm something has been stored first (empty DB returns nothing)
- Try a broader query

**Memory seems to forget things:**
- Check the importance value -- low-importance memories decay faster
- Recall the memory occasionally to boost its access count
- For critical facts, store with importance >= 0.8

**Slow recall:**
- Reduce `embeddingDimensions` to 512 if latency is a concern
- Ensure `dbPath` is on a fast disk (not a network mount)
