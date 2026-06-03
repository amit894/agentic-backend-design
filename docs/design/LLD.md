# LLD Output Index

This file is superseded by the `docs/design/problems/` folder structure.
Each problem now has its own subfolder. Do not write LLD output here.

---

## Active designs

| Problem | Folder | Status |
|---------|--------|--------|
| LRU Cache (basic — interview) | [`problems/lru-cache/`](./problems/lru-cache/) | Complete |
| LRU Cache (production — 100k RPS) | [`problems/lru-cache-production/`](./problems/lru-cache-production/) | Pending — run `/lld-round` |

---

## How to start a new design

1. Fill `docs/design/PROBLEM-BRIEF.md` with your problem.
2. Run `/lld-round` or `/design-and-ship` in Cursor.
3. The workflow creates `docs/design/problems/<problem-name>/lld.md` and writes output there.

See `.cursor/PIPELINE-DIAGRAM.md` for the full workflow diagram.
