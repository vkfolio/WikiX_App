# wikiX Launch Copy

> Generated 2026-05-04. All facts validated against `README.md` (engine repo).
> Accuracy notes inline where claims needed qualification.

---

## 1. Product Hunt Description

**Tagline:**
> A temporal knowledge graph for code, docs, and conversations — your agent's memory that knows when things changed.

**Body:**

Most knowledge tools answer queries from whatever is in the index. They cannot distinguish between a fact that is current and one that was superseded three sprints ago. A flat embedding index will confidently surface stale answers because it has no concept of when something was true.

wikiX is a self-hosted knowledge engine where every fact carries a validity window. `valid_at` and `invalid_at` timestamps are first-class on every entity, edge, and episode. Drop a folder of code, documentation, or chat logs and get back a queryable bi-temporal graph, a navigable wiki book auto-distilled from the graph topology, and a 13-tool MCP surface so coding agents can drive it directly.

**What makes it different:**

- **Bi-temporal graph.** Query "as of Q1" and get the knowledge graph as it actually was at that point. Contradictions invalidate old edges rather than erasing them. The timeline survives.
- **AST-first code extraction.** Tree-sitter for Python, TypeScript/TSX, JavaScript/JSX, Go, Rust, Java, and C/C++. Code symbols are extracted deterministically from the parser, not from an LLM guess. They carry `confidence=EXTRACTED` unconditionally.
- **Confidence taxonomy.** `EXTRACTED > INFERRED > AMBIGUOUS`. A lower-confidence claim that contradicts a higher-confidence fact emits a reviewable `ConflictEvent` instead of a silent overwrite.
- **13 MCP tools.** `search_wiki`, `code_lookup`, `call_graph`, `defined_in`, `mentions`, `find_similar_code`, `recent_changes`, `add_episode`, `graph_neighbors`, `list_corpora`, and more. Point Claude Code, Cursor, Codex, or Aider at it directly with a single `mcp.json` snippet.
- **Zero-friction self-host.** One Caddy ingress on port 8080, static bearer token generated on first boot, no login screens, no scopes, no multi-tenant complexity. `make up-host && make migrate` is the entire install path.

**Two products, one engine.**

The engine is Go + Python, MIT licensed, and open source. wikiX Desktop is the native daily-driver built on Tauri 2 and React 18: Cytoscape graph viewer with fcose and dagre layouts, syntax-highlighted book reader with Mermaid diagrams and KaTeX, streaming chat with tool-call cards, and an MCP Gateway for per-IDE token scoping.

**Where this came from.**

wikiX braids two independent lineages: Karpathy's April 2026 post on LLM-built wikis (which produced the graphify structural skeleton, AST-first extraction, and the confidence taxonomy) and Zep AI's graphiti (which contributed the bi-temporal episodic model and the `valid_at / invalid_at / expired_at` edge schema). Combining AST-first determinism with bi-temporal contradiction resolution and wrapping both in an MCP surface is the work this project adds on top.

---

## 2. Product Hunt First Comment (Maker Comment)

Hey Product Hunt.

I am Vignesh, and I built wikiX over nights and weekends starting in April 2026.

The motivation came from a specific and recurring problem: working in a codebase that had evolved significantly over six months, with design decisions spread across documents, Slack threads, and commit messages. Every time a coding agent answered a question about the system, it answered from whatever was in the embedding index, with no awareness of which version of a fact was current. RAG retrieves the most relevant chunks; it does not know when they stopped being accurate.

A temporal graph solves this differently. Every entity and edge carries `valid_at` and `invalid_at`. Contradictions invalidate old edges rather than overwriting them. Querying the graph "as of March 15" returns the world as it was on that date, not the latest version of everything.

Karpathy's April 2026 post provided the structural foundation: parse the folder, extract an entity and relation graph using an LLM, bucket the wiki chapters by Leiden community topology. Zep AI's graphiti provided the temporal model: each edge has `valid_at`, `invalid_at`, and `expired_at`, and contradictions are handled through invalidation rather than deletion. Combining those two lineages required solving six things neither project addresses:

1. Confidence-aware contradictions. graphiti's resolution pipeline has no confidence levels. An LLM-inferred edge can invalidate a tree-sitter-derived one if it arrives later. wikiX refuses that: claims below `EXTRACTED` that contradict ground truth become `ConflictEvent` rows for review.

2. Two-pool retrieval. Most hybrid systems merge chunks, entities, and edges into a single RRF-ranked list. wikiX keeps two separate pools (pgvector + tsvector for chunks; temporal graph for entities and edges), each with its own cross-encoder rerank pass, returned side-by-side.

3. The Neo4j Community Edition port. Stock graphiti targets per-`group_id` databases, which require Neo4j Enterprise. Community Edition has one user database. wikiX rebuilds the partitioning around property predicates, with a CI grep tripwire ensuring every Cypher query is routed through `with_group_id()`.

4. A transactional outbox with deterministic `episode_uuid` so an interrupted ingest resumes without duplicating or losing in-flight work.

5. AST-diff incremental updates. The graphify lineage caches at the file level. graphiti has no concept of file edits. wikiX walks the AST diff between old and new content and emits a delta episode that tombstones removed nodes and re-embeds only changed chunks.

6. Static bearer authentication. No login screens, no scopes, no token refresh. The token is generated on first boot, stored at `<dataDir>/token`, and never expires.

A few things I would especially appreciate feedback on today:

- **Install flow.** `make up-host && make migrate` should complete in under 30 seconds. If it does not, please open an issue with the service logs.
- **MCP tool surface.** 13 tools is a deliberate choice. I am curious whether the set feels complete for your agent workflows or whether a specific operation is missing.
- **Confidence taxonomy.** The `EXTRACTED > INFERRED > AMBIGUOUS` ordering reflects the determinism of the extraction method. Feedback on where the line should be drawn, or on cases where the taxonomy breaks down, is welcome.

Thank you for taking a look.

---

## 3. Reddit Post

**Recommended subreddits:** `r/LocalLLaMA` (primary), `r/selfhosted`, `r/programming`

**Title:**
> Built a self-hosted temporal knowledge graph for code, docs, and chats: bi-temporal facts, AST-first extraction, 13 MCP tools for coding agents [MIT, Go + Python]

**Body:**

TL;DR: **wikiX** is a self-hosted knowledge engine where every fact carries `valid_at` and `invalid_at` timestamps. Drop a folder of code, documentation, or chat logs and get a queryable bi-temporal graph, a wiki book auto-distilled from graph topology, and a 13-tool MCP surface for Claude Code, Cursor, Codex, and Aider. MIT licensed, single `make up-host`.

---

**Why temporal matters for code knowledge**

Standard vector search finds relevant chunks. It cannot tell you which version of a fact is current. If a design decision was reversed two sprints ago and both the old and new documents are in the index, the agent has no basis for preferring the current one. It returns whichever is most semantically similar to the query.

A temporal graph tracks when facts changed. Every entity and edge has `valid_at` and `invalid_at`. Contradictions invalidate old edges rather than overwriting them. Query at any past timestamp and get the world as it was.

---

**Origin**

Karpathy posted in April 2026 about compiling research notes into LLM-built wikis rather than re-running RAG over the same documents. He published a gist; within 48 hours `safishamsi/graphify` shipped a structural implementation: parse the folder, extract an entity and relation graph, bucket chapters by Leiden community. Several forks followed.

The temporal layer comes from `graphiti` (Zep AI). Each edge carries `valid_at`, `invalid_at`, and `expired_at`. Contradictions invalidate old edges. The two lineages are independent; graphiti predates the Karpathy wave entirely.

wikiX braids both: AST-first determinism from the graphify lineage, bi-temporal correctness from graphiti, and a 13-tool MCP surface so agents drive it directly rather than receiving prompt-stuffed context.

---

**How it differs from RAG over a repo**

| Dimension | Standard RAG | wikiX |
|---|---|---|
| Code extraction | LLM or regex | tree-sitter AST (7 language families) |
| Fact confidence | Implicit | `EXTRACTED > INFERRED > AMBIGUOUS` taxonomy |
| History | Overwritten | Every fact has `valid_at` + `invalid_at` |
| Contradictions | Silent overwrite | `ConflictEvent` row if higher-confidence fact is challenged |
| Retrieval | Single merged list | Two pools (chunks + graph), each cross-encoder reranked |
| Agent interface | Prompt stuffing | 13 structured MCP tools |

---

**The 13 MCP tools**

```
search_wiki         hybrid retrieval: chunks + entities + edges
code_lookup         symbol → entity + 1-hop neighbors + source span
call_graph          upstream callers + downstream callees, N hops
file_neighbors      every entity in a file + cross-file edges
defined_in          "where is X defined?" with snippet inline
mentions            every chunk + entity referencing a symbol
find_similar_code   embedding-similar functions and classes
find_similar_images multimodal similarity for diagrams in PDFs
recent_changes      episodes added since a given timestamp
add_episode         upsert a fact with bi-temporal correctness
read_episode        fetch an episode by UUID
graph_neighbors     N-hop subgraph from a center entity
list_corpora        what is indexed in the current wiki
```

Drop the `mcp.json` snippet from `mcp/skills/` into your IDE config and the agent gains structured access to the entire graph rather than relying on approximate retrieval.

---

**Stack**

Go (Gin) for the REST and admin services. Python (FastAPI) for the agent, extractor, docreader, reranker, and graph-worker. Postgres with pgvector and tsvector for chunk retrieval. Neo4j Community 5.26 for the temporal graph. Redis for the Asynq task queue. MinIO for object storage. Caddy as the single-port ingress. Desktop app is Tauri 2 with React 18 and Cytoscape.js.

`make up-host` bundles Neo4j, MinIO, and Jaeger and expects host Postgres and Redis. `make up` bundles everything including a paradedb/paradedb Postgres image.

---

**The Neo4j Community Edition problem**

Stock graphiti requires per-`group_id` database isolation, which is a Neo4j Enterprise feature. Community Edition has one user database. wikiX rebuilds the partitioning around property predicates: every Cypher query is routed through a `with_group_id(query, gid)` helper enforced by a CI grep tripwire. Forks that skip this will silently leak data across wikis. This took two weekends to get right and is not documented anywhere in the graphiti codebase.

If you are building on graphiti and have not hit this yet, you will.

---

**Links**

- Engine (MIT): `github.com/vkfolio/wikiX`
- Desktop app: `github.com/vkfolio/WikiX_App`
- Launch page: `wikix.vkfolio.com`

Happy to answer questions about the bi-temporal model, the confidence taxonomy, the Neo4j Community port, or integration with a specific coding agent.

---

## 4. X / Twitter Post

### Option A: Single tweet

> Standard RAG finds the chunks. It cannot tell you when they stopped being true.
>
> wikiX: every fact has `valid_at` and `invalid_at`. Query your codebase "as of Q1" and get the graph as it actually was.
>
> 13 MCP tools. Tree-sitter AST extraction. Self-hosted, MIT.
>
> github.com/vkfolio/wikiX

---

### Option B: Thread (recommended for launch day)

**Tweet 1 — hook:**
> RAG retrieves the most relevant chunks. It has no concept of when those chunks stopped being accurate.
>
> wikiX is a self-hosted temporal knowledge graph: every fact carries `valid_at` and `invalid_at`. Query your codebase at any past timestamp and get the world as it was.
>
> Just shipped. MIT. 🧵

**Tweet 2 — origin:**
> This braids two independent lineages.
>
> @karpathy's April 2026 post on LLM-built wikis produced graphify: parse folder, extract entity graph, bucket wiki chapters by topology.
>
> @ZepAI's graphiti contributed the bi-temporal model: `valid_at`, `invalid_at`, `expired_at` on every edge. Contradictions invalidate rather than overwrite.

**Tweet 3 — differentiators:**
> What wikiX adds on top:
>
> Code symbols from tree-sitter (7 language families), not LLM inference. `confidence=EXTRACTED` cannot be overwritten by an `INFERRED` claim.
>
> Two retrieval pools (chunks and graph), each cross-encoder reranked, returned side-by-side rather than blindly merged.

**Tweet 4 — agent integration:**
> 13 MCP tools for coding agents.
>
> `search_wiki("event sourcing")`
> 3 chunks · 1 entity · valid as of 2026-04-21
>
> `call_graph("ProcessOrder")`
> upstream callers + downstream callees + source spans
>
> One `mcp.json` snippet. Works with Claude Code, Cursor, Codex, and Aider.

**Tweet 5 — CTA:**
> Engine: MIT, `make up-host` (Go + Python + Postgres + Neo4j)
> Desktop: Cytoscape graph viewer, syntax-highlighted book, MCP Gateway for per-IDE token scoping
>
> github.com/vkfolio/wikiX
> wikix.vkfolio.com
>
> Architecture notes and Q&A in the first comment on Product Hunt.

---

## Accuracy Notes

All claims validated against `README.md` (engine repo) on 2026-05-04.

- **13 MCP tools**: confirmed — search_wiki, code_lookup, call_graph, file_neighbors, defined_in, mentions, find_similar_code, find_similar_images, recent_changes, add_episode, read_episode, graph_neighbors, list_corpora.
- **7 language families**: confirmed — Python, TypeScript/TSX, JavaScript/JSX, Go, Rust, Java, C/C++.
- **Stack**: confirmed — Go, Python, Postgres, pgvector, tsvector, Neo4j Community 5.26, Redis, MinIO, Caddy, Tauri 2, React 18, Cytoscape.js.
- **`make up-host && make migrate`**: confirmed as the install path.
- **Karpathy post date**: April 2026, confirmed in README.
- **graphify author**: `safishamsi/graphify`, confirmed in README.
- **graphiti**: Zep AI, Apache-2.0, confirmed in README.
- **Port**: 8080, confirmed (Caddy ingress).
- **Neo4j Enterprise / Community partitioning issue**: confirmed in README under "What we added on top."
- **ConflictEvent**: confirmed under ADR 0002 reference in README.
- **`with_group_id()` CI tripwire**: confirmed in README.
- **MIT license for engine only**: confirmed — Desktop ships under a separate license from `github.com/vkfolio/WikiX_App`.
