<h1 align="center">wikiX Desktop</h1>
<h2 align="center">The polished daily-driver for the wikiX knowledge engine</h2>

<div align="center">

[![Latest release](https://img.shields.io/github/v/release/vkfolio/WikiX_App?label=Latest&color=22c55e)](https://github.com/vkfolio/WikiX_App/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/vkfolio/WikiX_App/total?color=2563eb)](https://github.com/vkfolio/WikiX_App/releases)
[![Engine: MIT](https://img.shields.io/badge/engine-vkfolio%2FwikiX-22d3ee)](https://github.com/vkfolio/wikiX)

</div>

> [!IMPORTANT]
> This repo hosts the **signed installer downloads only**. App source code lives in a separate repo. The MIT-licensed knowledge engine the desktop talks to is **[vkfolio/wikiX](https://github.com/vkfolio/wikiX)** — install that first.

wikiX Desktop turns a folder of code, documents, and PDFs into a queryable
knowledge graph + chat interface. Drop in a folder, ask questions, get
answers with citations. Cytoscape graph viewer, syntax-highlighted Book
reader, streaming chat with tool-call cards, smart corpus folder watcher,
MCP Gateway for Claude Code / Cursor / Codex / Aider, and an Insights
dashboard with god-nodes + surprising connections.

---

## Download

Pick the latest installer from **[Releases](https://github.com/vkfolio/WikiX_App/releases/latest)**:

| OS | File | Notes |
|---|---|---|
| **macOS** (Apple Silicon + Intel) | `wikiX-Desktop_<version>_universal.dmg` | Drag into `/Applications` |
| **Windows 10/11** (x64) | `wikiX-Desktop_<version>_x64.msi` | Run the MSI; admin prompt may appear |
| **Linux** (x64, AppImage) | `wikix-desktop_<version>_amd64.AppImage` | `chmod +x` then double-click |
| **Linux** (Debian / Ubuntu) | `wikix-desktop_<version>_amd64.deb` | `sudo dpkg -i wikix-desktop_*.deb` |

Auto-update is built in — once installed the app pulls future releases
straight from this repo.

---

## Before you install

You also need the **wikiX engine** running somewhere reachable. The
desktop is the UI; the engine is the brain.

```bash
# In a separate terminal:
git clone https://github.com/vkfolio/wikiX.git
cd wikiX
make up-host
make migrate
cat $WIKIX_DATA_DIR/token   # remember this token
```

If you're starting fresh, follow the **[engine quickstart](https://github.com/vkfolio/wikiX/blob/main/docs/quickstart.md)** end-to-end, then come back here.

---

## First run

1. Launch wikiX Desktop.
2. The first-run wizard appears. The **Engine URL** is pre-filled
   (`http://localhost:8080`). The **Bearer token** is auto-filled if the
   engine is running on the same machine; otherwise paste the token from
   `$WIKIX_DATA_DIR/token`.
3. Click **Connect**. Welcome to wikiX.

Once connected:

- **Corpus** — pick a folder. Smart-skip ignores `node_modules`, `.venv`,
  `__pycache__`, etc. by default. The pre-sync forecast warns before
  expensive embed runs.
- **Search** — full SearchRequest builder with hybrid / chunks / graph /
  code modes, confidence floor, k-knobs, rerank toggle, and saved views.
- **Graph** — Cytoscape force-directed viewer with five layouts;
  confidence-styled borders (solid / dashed / dotted) and greyed
  invalidated edges. PNG + JSON export.
- **Chat** — full SSE event handling. Tool calls render as collapsible
  cards with args + result + latency. Markdown rendering with shiki
  code highlighting, Mermaid diagrams, and KaTeX math. Per-turn
  retrieval inspector.
- **Book** — auto-distilled chapters with `[[wikilinks]]` and
  `*[cite needed]*` pills.
- **MCP Gateway** — generate per-IDE config snippets to point Claude
  Code / Cursor / Codex / Aider at this engine.
- **Insights** — god-nodes, surprising connections, recent activity.
- **Atlas** — cross-wiki bubble map.

---

## Verify the download

Each release ships SHA-256 checksums and platform-native signatures.

- **macOS**: notarised by Apple. macOS verifies on launch — if Gatekeeper
  blocks the first open, right-click the app → *Open* once.
- **Windows**: Authenticode-signed. Windows SmartScreen may show a
  one-time warning until the cert builds reputation.
- **Linux**: AppImage is detached-GPG-signed. Verify with:
  ```bash
  gpg --verify wikix-desktop_*.AppImage.sig wikix-desktop_*.AppImage
  ```

Checksums live alongside each artifact:
```bash
sha256sum -c wikix-desktop_<version>_SHA256SUMS.txt
```

---

## Troubleshooting

**"Couldn't reach the engine"** — `docker ps` should list the wikiX
containers as `Up`. If not, in your engine clone: `make up-host`.

**"Token rejected"** — `cat $WIKIX_DATA_DIR/token` and paste freshly. The
token is generated on first engine boot and never expires; if you
re-bootstrapped the engine you got a new one.

**"Search returns nothing after upload"** — embeddings need an LLM
provider. Easiest path: install [Ollama](https://ollama.com), then
`ollama pull nomic-embed-text` and configure it in **Settings →
Providers → Embedding**. The desktop's first-run wizard auto-detects
Ollama and recommends models.

**"App won't open on macOS — `developer cannot be verified`"** — first
launch only: right-click the app, select *Open* from the context menu,
confirm the warning dialog. Subsequent launches work normally.

**"Linux AppImage won't execute"** — `chmod +x wikix-desktop_*.AppImage`.

---

## What's the relationship to the engine?

| Repo | What it is | Visibility |
|---|---|---|
| [**vkfolio/wikiX**](https://github.com/vkfolio/wikiX) | The MIT-licensed knowledge engine — REST API, MCP server, ingest pipeline, retrieval, book distillation. Self-host with `make up-host`. | Public |
| **vkfolio/WikiX_App** *(this repo)* | Signed installers + auto-update channel for wikiX Desktop. | Public |
| **vkfolio/wikiX-desktop** | Source code for the desktop app. | Private |

The desktop talks to the engine **only through the engine's public REST
API**. Anyone can build a different client against the same engine.

---

## Reporting issues

- **Engine bugs / API issues** → file on
  [vkfolio/wikiX/issues](https://github.com/vkfolio/wikiX/issues).
- **Desktop bugs / install issues / UI feedback** → file here on
  [vkfolio/WikiX_App/issues](https://github.com/vkfolio/WikiX_App/issues).
- **Security** — please don't open public issues; email the maintainer
  privately and we'll coordinate disclosure.

---

## Release cadence

Tagged releases follow `vMAJOR.MINOR.PATCH`. Each tag triggers a build
matrix that produces all four artifacts (`.dmg`, `.msi`, `.AppImage`,
`.deb`) plus checksums and detached signatures, attached to the GitHub
Release. Auto-update consults
`https://github.com/vkfolio/WikiX_App/releases/latest/download/latest.json`.

Release notes summarise: what changed, what broke, what's compatible
with which engine version. The desktop refuses to start against an
engine more than one minor version behind to keep the integration
surface honest.

---

## License

Distributed under the wikiX Desktop EULA — see `LICENSE` in each release
archive. The underlying knowledge engine is MIT — see
[vkfolio/wikiX/LICENSE](https://github.com/vkfolio/wikiX/blob/main/LICENSE).

## Acknowledgements

The whole stack started with
[**Andrej Karpathy's April 2026 post**](https://x.com/karpathy/status/2039805659525644595)
on LLM-built knowledge bases — the engine that powers this desktop is one
downstream branch of the wave that followed. Direct conceptual debts go
to [graphify](https://github.com/safishamsi/graphify) (structural layer)
and [graphiti](https://github.com/getzep/graphiti) (the bi-temporal
layer, separate lineage from Zep AI). Plus the long list of libraries
catalogued in the
[engine README](https://github.com/vkfolio/wikiX#libraries--frameworks-that-make-this-work) —
Tauri, React, Cytoscape.js, marked, DOMPurify, Shiki, Mermaid, KaTeX,
FastAPI, Gin, Asynq, pgvector, Neo4j, Caddy, OpenTelemetry, tree-sitter,
and the rest.
