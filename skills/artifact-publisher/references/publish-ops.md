# Publish operations

Use the connected Artifact Publisher MCP. Do not shell out, curl the endpoint,
or wrap tools in scripts.

## Publish

1. If a real HTML file exists on the machine, **read it** and pass the full
   string as `html`. Do not invent a stub or paraphrase.
2. HTML must be self-contained: sandbox has **no network**, so inline CSS,
   scripts, fonts, and images (e.g. data URIs). Plain links still open in a
   new tab for the viewer.
3. Optional before large styling: `get_authoring_guide`. Prefer
   `<meta name="ap-theme" content="firn">` in `<head>`.
4. Call `publish_artifact` with `html` and optional `title`, `summary`,
   `expires_at`, `comments_enabled`, `visibility`, `allowed_emails`.
5. On success, report **`artifact.url` first** (control-origin share/viewer).
   Add `artifact.viewer_url` when a version pin helps. Do **not** lead with the
   raw artifact-origin document URL (iframe-only).

Default visibility is `public` (unlisted by link; not indexed). For coworkers
without accounts, use `public`, not `allowlist`.

Free plan (current product defaults): size ceiling on UTF-8 HTML (see tool
error / docs; typically 10 MiB), 5 active artifacts, 50 versions — treat live
tool errors as authoritative.

## Update (same link)

1. Call `update_artifact` with `artifact_id` (id, slug, or viewer URL) and
   `expected_version` equal to the current version.
2. Pass new `html` for a new immutable version; the share link moves to it.
   Earlier versions stay at pinned URLs.
3. Optional `title` renames; title-only (no `html`) renames without a new body
   version (still counts toward some rate limits — see tool text).
4. Again report **`artifact.url` first**.

## Access without a new version

`set_artifact_access` — change `visibility` / `allowed_emails` without bumping
the HTML version.

| visibility | Who can open |
| --- | --- |
| `public` (default; `unlisted` alias) | Anyone with the link; no sign-in |
| `private` | Owner only (signed in) |
| `allowlist` | Signed-in users whose primary email is listed |

## Feedback (optional, secondary)

Not an approval workflow (share-first product).

- `get_feedback` — pull comments (oldest first). Filters: `unresolved_only`,
  `since`, `version` (default current for `anchor_status`), page with `limit` /
  `cursor`.
- `reply_to_feedback` — one reply per comment; optional `resolve: true`.
  Second reply → conflict.

Selected-passage / quote-anchored comments may appear when the viewer supports
them; do not claim long-press or chat-native annotation UX.

## List / revoke

- `list_artifacts` — workspace artifacts, newest first, unresolved counts.
- `revoke_artifact` — stop serving (410); cannot undo. Confirm with the user.

## Shared state

Synced JSON key/value for page JS (`ArtifactRelayState`). Limits: 200 keys,
16 KiB/value, 64 KiB total.

- `get_artifact_state` / `set_artifact_state` / `reset_artifact_state`
- Deprecated: `get_artifact_board` / `reset_artifact_board` — prefer state tools

## Workspace defaults

`configure_workspace` — read/update allowed defaults (empty settings = read).
Platform guardrails are refused. Not available in every workspace.

## Authoring

`get_authoring_guide` — themes, `ap-*` layout classes, example, snippet, size
and sandbox rules. Optional `theme` (`firn`, `firn@1`) validated as selected.

## Non-goals

Multi-file / SPA hosting, DOM annotation bundles, local server
install/upgrade/backup, inventing HTML when a file exists.
