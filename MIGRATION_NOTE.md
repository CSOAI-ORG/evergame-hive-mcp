# MIGRATION_NOTE - MCP 2026-07-28 wire - class `header-add`

**Date:** 2026-10-08 - **Lane:** M4 MCP-migration (header-add wave 2, batch 4) - **Branch:** `mcp-2026-wire-header-add`
**Runbook:** `MCP_2026_WIRE_MIGRATION_PLAN_2026-10-07.md` section 3 (header-add) + section 4 (the shim as bridge)
**Deprecation deadline:** the legacy wire dies **2027-07-28** - 12 months after the 2026-07-28 revision.

## 1. Transport reality

This server is **Node/TypeScript** (`src/index.ts`, entry `dist/index.js`), manifest is
`package.json` only - there is no `pyproject.toml` and no `server.py`. It runs on stdio.

## 2. What changed in this branch (and what deliberately did not)

1. `mcp2026_shim.py` vendored at the repo root (Python stdlib, zero third-party deps) so the
   2026-07-28 ingress middleware is in-tree and deployable in front of this server.
2. **No `@modelcontextprotocol/sdk` pin was written.** The real dependency is
   `"@modelcontextprotocol/sdk": "^1.15.0"`; no published JS SDK release speaks the
   2026-07-28 wire, and this lane does not invent JS pins. The dependency line is therefore
   left exactly as it was.
3. **No migration-note comment block was added to `src/index.ts`.** The Python pattern
   (note block + `http_app()` enable path in `server.py`) has no counterpart here, and
   editing a TypeScript entry file with a Python-only shim available would misrepresent
   what is actually wired.

The vendored shim is Python: it cannot run inside a Node process. It applies where plan
section 4 says it does - **one shim instance per ingress (reverse proxy / gateway)**. Until
that ingress exists, `evergame-hive-mcp` still speaks the legacy wire.

## 3. Verify

```bash
PYTHONPATH= /opt/homebrew/bin/python3.11 ~/clawd/mcp_wire_audit.py audit --local evergame-hive-mcp
```

| state | era | migration |
|---|---|---|
| before (default branch) | unknown | header-add |
| **after (this branch)** | **unknown** | **header-add** |
| control (note text removed) | unknown | header-add |

All three rows are identical on purpose: this branch adds documentation and the vendored
shim only, so the static scan has nothing new to read. Files changed in this branch:
`mcp2026_shim.py`, `MIGRATION_NOTE.md`. The scanner skips `mcp2026_shim.py` by design
(`SELF_FILES`) and does not scan `.md`.

**Honest reading:** `after` == `before` == `control` here. This PR is **not** evidence that
`evergame-hive-mcp` speaks 2026-07-28. It records the class, the blocker (no JS SDK release speaks the
wire) and the placement of the bridge.

## 4. Follow-ups (not in this branch)

* A Node-capable ingress shim, or an upstream `@modelcontextprotocol/sdk` release speaking
  2026-07-28, before any claim of 2026-07-28 support for this server.
* Static declaration surfaces (`.well-known/*.json`, `server.json`, `smithery.yaml`,
  `README.md`) still declare an older wire - follow-ups, not silent-edited (Art. 21: a
  declaration change gets its own commit).
* A live probe per plan section 6.5 is owed before anyone quotes `migration: none`.

Plan: `MCP_2026_WIRE_MIGRATION_PLAN_2026-10-07.md` - deadline 2027-07-28 - measurement, not certification.
