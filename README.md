# Artifact Publisher — Cursor plugin (v0.1.0)

**Status:** scaffold only. Do **not** submit to Cursor Marketplace until Bobby/Simon go.

Cursor Plugin pack that wires the hosted Artifact Publisher MCP
(`https://app.artifactpublisher.com/mcp`) plus the portable Agent Skill.

## Layout

```text
plugins/cursor/
├── .cursor-plugin/plugin.json   # Cursor Plugin manifest
├── mcp.json                     # Remote MCP (OAuth via discovery)
├── skills/artifact-publisher/   # Copy of canonical skills/artifact-publisher/
├── README.md
└── LICENSE                      # MIT-0
```

Canonical skill source of truth: `skills/artifact-publisher/` (keep in sync;
`scripts/sync-skill-packs` is a follow-up).

## Auth: OAuth preferred

`mcp.json` declares **only** the remote URL — no `headers`, no API key.

```json
{
  "mcpServers": {
    "artifact-publisher": {
      "url": "https://app.artifactpublisher.com/mcp"
    }
  }
}
```

Cursor discovers OAuth (PKCE) from the server’s protected-resource / AS metadata
and opens browser consent — same preferred path as ChatGPT Apps / Claude
Connect. Complete consent in the browser; never paste access tokens into chat.

### Advanced: publisher key

For headless / always-on clients that cannot do browser OAuth, mint a key at
https://artifactpublisher.com/account/ and set `ARTIFACT_PUBLISHER_KEY`, then
use a **local** Cursor MCP override (not this pack’s committed `mcp.json`):

```json
{
  "mcpServers": {
    "artifact-publisher": {
      "url": "https://app.artifactpublisher.com/mcp",
      "headers": {
        "Authorization": "Bearer ${env:ARTIFACT_PUBLISHER_KEY}"
      }
    }
  }
}
```

Never commit `ar_pk_…` values. Prefer OAuth when the IDE supports it.

## Local install / test (before Marketplace)

1. Open Cursor → **Settings → MCP** (or Customize / Plugins).
2. Either:
   - Point a local/dev plugin path at this folder (`plugins/cursor/`), or
   - Copy `mcp.json`’s server entry into `~/.cursor/mcp.json` / `.cursor/mcp.json`
     and enable the server; click Connect / Login for OAuth.
3. Confirm tools appear (`publish_artifact`, `list_artifacts`, …).
4. Optional: ensure the skill under `skills/artifact-publisher/` is visible to
   the agent (plugin skill discovery).

## Format notes (vs ChatGPT Apps pack)

| | This Cursor pack | ChatGPT Apps (`docs/chatgpt-plugin/` + growth ZIP) |
| --- | --- | --- |
| Manifest | `.cursor-plugin/plugin.json` (Cursor Plugin) | Root `plugin.json` (Agent Plugins / OpenAI extensions) |
| MCP auth | URL only → Cursor OAuth discovery | `auth: "oauth"` or Agent Plugins `extensions.com.openai.auth` (CIMD) |
| Skills | Bundled under `skills/` | Often link-only ZIP without skills |
| Listing copy | Growth owns Marketplace listing text / screenshots | Growth owns Apps directory copy |

Agent Plugins root `plugin.json` + `mcp.json` also loads in Cursor, but this
pack uses the **Cursor Plugin** layout so Marketplace expectations (logo,
keywords, skill discovery) match Cursor docs.

## Do not publish

- No Marketplace submit (`cursor.com/marketplace/publish`) until Bobby go.
- No ClawHub / Hermes publish from this tree.
- Growth owns listing copy, screenshots, and submit drafting.

## Related

- Canonical skill: `skills/artifact-publisher/`
- ClawHub pack (separate): `plugins/clawhub/artifact-publisher/`
- ClawHub checklist: `docs/internal/skill-clawhub-checklist.md`
- ChatGPT MCP pattern: `docs/chatgpt-plugin/mcp.json`
- ADR 0025: `docs/decisions/0025-portable-agent-skill-and-connect.md`
