# Failures

Load after an auth, size, quota, or consent failure. Do not invent success.

## Authentication

| Symptom | What to do |
| --- | --- |
| `401` / missing or bad Bearer | Confirm MCP config uses env `ARTIFACT_PUBLISHER_KEY` or OAuth client store. Point user to https://artifactpublisher.com/account/ to mint/rotate. Never ask them to paste the key or `ar_oat_…` into chat. |
| Keyless call to a write/read tool | Only `initialize`, `tools/list`, and `ping` are keyless. Publish and other tools need a credential. |
| OAuth consent abandoned / expired | Ask the user to complete browser consent again in their MCP client. Do not collect tokens in chat. |
| Wrong workspace / `not_found` on `artifact_id` | Id, slug, or viewer URL must belong to this key’s workspace. Re-check with `list_artifacts`. |

Never fall back to printing Authorization headers, cookies, or raw secrets “to debug.”

## Size and HTML validation

| Symptom | What to do |
| --- | --- |
| Size / UTF-8 limit error | Shrink HTML; inline only what is needed. Live error text is authoritative for the workspace ceiling. |
| `invalid_input` on theme / HTML | Fix per error message (e.g. `ap-theme` marker). Call `get_authoring_guide` rather than inventing CSS. |
| Sandbox “broken” assets | External fonts/images/APIs will not load — inline or drop them. |

## Quota

| Symptom | What to do |
| --- | --- |
| `quota_exceeded` on active artifacts | Revoke or let expire unused artifacts (`revoke_artifact`), or ask the user to manage plan at the desk/pricing page. Do not invent upgrade checkout flows. |
| `quota_exceeded` on versions | Prefer fewer updates; revoke old artifacts if appropriate. Free plan version caps apply — see error details. |
| Rate limit / `429` | Wait for `Retry-After` if present; retry once. Do not hammer the endpoint. |

## Update conflicts

| Symptom | What to do |
| --- | --- |
| `expected_version` mismatch | Another publish won the race. Re-read current version (`list_artifacts` / prior result), explain, and ask before retrying with the user’s HTML. |

## Maintenance / unavailable

If the server returns maintenance or read-only errors: report plainly; prefer
read tools (`get_authoring_guide`, `list_artifacts`, `get_feedback`) when those
still work. Do not switch to a different product or invent a local host.

## Safety

Stop on uncertainty. Do not download or run binaries, curl wrappers, or
“helper scripts” to work around MCP failures.
