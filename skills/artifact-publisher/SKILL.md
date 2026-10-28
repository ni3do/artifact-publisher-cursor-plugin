---
name: artifact-publisher
description: Publish self-contained HTML to a stable share link and read page feedback via Artifact Publisher MCP. Use when the user wants a review/share URL for agent-made HTML, to update the same link, change access, pull comments, or style with built-in themes. Do not use for unrelated hosting, SPA/multi-file deploys, or package registries.
version: 0.1.0
author: Angle Labs GmbH
license: MIT-0
platforms: [macos, linux, windows]
required_environment_variables:
  - name: ARTIFACT_PUBLISHER_KEY
    optional: true
    prompt: "Publisher key for headless/always-on agents (Hermes, OpenClaw operators). Mint once at https://artifactpublisher.com/account/ — shown once; never paste into chat. Prefer OAuth browser consent when the client supports it."
metadata:
  tags: [html, publishing, sharing, share-link, mcp, feedback, review]
  hermes:
    tags: [html, publishing, sharing, mcp, feedback]
    homepage: https://artifactpublisher.com
  openclaw:
    emoji: 📎
    homepage: https://artifactpublisher.com
    primaryEnv: ARTIFACT_PUBLISHER_KEY
    requires:
      env:
        - ARTIFACT_PUBLISHER_KEY
    envVars:
      - name: ARTIFACT_PUBLISHER_KEY
        required: true
        description: Scoped publisher key (Bearer). Prefer OAuth when the client supports browser consent; for headless/always-on agents this env var is the practical path. Mint at https://artifactpublisher.com/account/ — never commit or paste into chat.
---

# Artifact Publisher

Hosted MCP server that turns one self-contained HTML document into a stable,
unlisted share link. Reviewers open it without an AI-vendor account; optional
page comments come back to the agent with `get_feedback`. MCP-first and
provider-agnostic. Free plan available.

**Server:** `artifact-publisher` · version `0.1.0`
**Endpoint:** `https://app.artifactpublisher.com/mcp` (streamable HTTP)
**Docs:** https://artifactpublisher.com/docs/mcp/

## Hard rules

1. After `publish_artifact` / `update_artifact`, report **`artifact.url`**
   (control-origin share/viewer) **first**. Use `artifact.viewer_url` when a
   version pin helps. **Never** make the raw artifact-origin URL the primary
   share link.
2. **Never** print, paste, commit, or write into artifacts: `ar_pk_…`,
   `ar_oat_…`, `ar_ort_…`, `ar_oac_…`, refresh tokens, cookies, or
   `Authorization` headers. Prefer OAuth or env `ARTIFACT_PUBLISHER_KEY`.
3. Do **not** invent HTML when a real file exists — read the file and pass its
   contents to `publish_artifact` / `update_artifact`.
4. Call `get_authoring_guide` before large styling; prefer
   `<meta name="ap-theme" content="firn">`.
5. Feedback is optional/secondary. `get_feedback` is **pull-only** — do not
   frame the product as an approval loop.
6. **Thin MCP wiring only** — no shell scripts, no curl wrappers, no reckless
   `exec`, no downloading or running arbitrary binaries.

## Route the request

| Route | When | Load |
| --- | --- | --- |
| Publish / update / access / feedback / state / authoring | Publish HTML, refresh a link, read/reply to feedback, themes, shared state | [publish-ops](references/publish-ops.md) |
| Desk / keys / account links | User asks to sign in, mint/revoke keys, or connect MCP from the desk | [desk-admin](references/desk-admin.md) |
| Failures | Auth, size, quota, or consent problems | [failures](references/failures.md) only after a failure |

## Setup

Wire the hosted MCP once. This skill is markdown + judgement only; it does not
ship binaries.

### 1. Credential

1. Open https://artifactpublisher.com/account/ and mint a publisher key (shown
   once). Store it in the client secret store or env — **never echo it in chat**.
2. **OAuth (preferred when the client supports browser consent):** register
   `https://app.artifactpublisher.com/mcp`, complete consent in the browser;
   never ask the user to paste an access token into chat.
3. **Key (practical for Hermes / OpenClaw headless / always-on):** set
   `ARTIFACT_PUBLISHER_KEY` and send `Authorization: Bearer ${ARTIFACT_PUBLISHER_KEY}`.
4. Keyless `initialize` / `tools/list` / `ping` work for discovery only.

### 2. Hermes Agent

```bash
hermes mcp add artifact-publisher --url https://app.artifactpublisher.com/mcp
```

When asked about auth, answer yes for publish tools and store the key in
`~/.hermes/.env` (do not leave it only in a shell export the gateway never sees).

Hand-edited `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  artifact-publisher:
    url: "https://app.artifactpublisher.com/mcp"
    headers:
      Authorization: "Bearer ${ARTIFACT_PUBLISHER_KEY}"
```

Reload with `/reload-mcp`. Keep the key out of chat and out of `SKILL.md`.

### 3. OpenClaw

```bash
openclaw mcp set artifact-publisher '{"url":"https://app.artifactpublisher.com/mcp","transport":"streamable-http","headers":{"Authorization":"Bearer ${ARTIFACT_PUBLISHER_KEY}"}}'
openclaw mcp list
```

Or in OpenClaw config (typical `~/.openclaw/config.json`):

```json
{
  "mcp": {
    "servers": {
      "artifact-publisher": {
        "url": "https://app.artifactpublisher.com/mcp",
        "transport": "streamable-http",
        "headers": {
          "Authorization": "Bearer ${ARTIFACT_PUBLISHER_KEY}"
        }
      }
    }
  }
}
```

**Registry publish is not done yet** (no ClawHub / Hermes public tap /
skills.sh). Install from the public docs page
https://artifactpublisher.com/docs/skill/ or copy the in-repo tree
`skills/artifact-publisher/` until a registry publish is greenlit.

### 4. Other MCP clients

**Claude Code** (prefer env-backed header; do not paste the key into the command
history if you can avoid it):

```sh
claude mcp add --transport http artifact-publisher https://app.artifactpublisher.com/mcp \
  --header "Authorization: Bearer ${ARTIFACT_PUBLISHER_KEY}"
```

**Cursor / Codex / similar** (`mcpServers`):

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

(Some clients use `${ARTIFACT_PUBLISHER_KEY}` without `env:` — match your client.)

### 5. Desk

Mint and manage keys at https://artifactpublisher.com/account/. Keys are shown
once; only a hash is stored server-side.

## Live tools (honesty)

Only these names exist on the live surface today
(`ARTIFACT_RELAY_TOOLS` / `tools/list`):

`publish_artifact`, `update_artifact`, `set_artifact_access`, `get_feedback`,
`reply_to_feedback`, `list_artifacts`, `revoke_artifact`, `configure_workspace`,
`get_artifact_board`, `reset_artifact_board`, `get_artifact_state`,
`set_artifact_state`, `reset_artifact_state`, `get_authoring_guide`.

(`get_artifact_board` / `reset_artifact_board` are deprecated aliases — prefer
the state tools.)

## Quick loop

1. Read existing HTML if a file exists; else author self-contained HTML (inline
   assets; sandbox has no network).
2. Optional: `get_authoring_guide` + `ap-theme` firn before large CSS.
3. `publish_artifact` → report **`artifact.url` first**.
4. Later: `update_artifact` (same link), optional `get_feedback` /
   `reply_to_feedback`, `set_artifact_access`, or `revoke_artifact`.

Details: [publish-ops](references/publish-ops.md).
