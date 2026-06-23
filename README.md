# Logos Tax Systems — Architecture Docs

_Last updated: 2026-06-23_

| Document | Read when |
|----------|-----------|
| **[Platform System Map](./platform-system-map.md)** | **All repos, features, modules, connections — the full Mermaid diagram** |
| [Architecture Viewer](./viewer.html) | Browser view — **Full System Map** tab + focused diagrams |
| [Product Architecture](./product-architecture.md) | Product decisions, UX, five pillars, user journeys, engagement flow |
| [Technical Architecture](./technical-architecture.md) | Repo boundaries, APIs, AI/RAG layer, deployment, infrastructure |
| [Business Context](../projects/logos-platform/context/business.md) | Pitch, GTM, business rules, engagement process detail |
| [Active Projects](../context/active-projects.md) | Current phase, shipped work, next actions |

## Viewing architecture

**Live site:** https://humanautics.github.io/logos-architecture/

**Local:** open [`viewer.html`](./viewer.html) or [`index.html`](./index.html) in any browser:

```bash
open architecture/viewer.html
```

Or serve locally if you prefer a URL:

```bash
npx --yes serve architecture -p 3456
# http://localhost:3456/
```

Tabs: **Full System Map** (all repos + connections) · Platform Overview · Focused Diagrams · Product · Technical · Open Gaps.

Requires network on first load (Mermaid CDN). Scroll/zoom on the full system map as needed.

## Deploying updates

This folder is published to **GitHub Pages** from [HUMANAUTICS/logos-architecture](https://github.com/HUMANAUTICS/logos-architecture). Push to `main` to redeploy (workflow copies `viewer.html` → `index.html` automatically).

```bash
cd architecture
git add viewer.html README.md platform-system-map.md product-architecture.md technical-architecture.md taxpert-architecture.md
git commit -m "Update architecture docs"
git push origin main
```

## History

`taxpert-architecture.md` was a single monolithic technical diagram (last updated 2026-06-09). It was split on 2026-06-23 into product + technical docs to reflect the five-pillar product model and recent shipping work.

## Dev Copilot RAG

Architecture files are indexed in TT-dev-copilot's codebase vector search. After significant updates, re-ingest:

```bash
cd TT-dev-copilot
python data_ingestion/upload_docs_direct.py
```
