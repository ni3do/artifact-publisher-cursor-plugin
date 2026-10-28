# Desk / account / connect

Load only when the user explicitly asks to sign in, mint or revoke keys, manage
the account, or connect MCP from the desk. Do **not** invent Stripe customer
IDs, key plaintext, plan prices, or checkout steps.

## Desk URLs

| Need | Link |
| --- | --- |
| Account / mint publisher key | https://artifactpublisher.com/account/ |
| MCP docs | https://artifactpublisher.com/docs/mcp/ |
| Quickstart | https://artifactpublisher.com/docs/ |
| Agent Skill | https://artifactpublisher.com/docs/skill/ |
| Pricing (user opens themselves) | https://artifactpublisher.com/pricing/ |
| Privacy | https://artifactpublisher.com/privacy/ |
| Imprint / operator | https://artifactpublisher.com/imprint/ |
| Support | contact@artifactpublisher.com |

A desk `#mcp` deep-link may land later; until then, point people at **Account**
plus MCP docs. Do not invent fragment URLs.

## Keys

1. User mints a scoped publisher key in the desk UI.
2. Key is **shown once**; server stores only a hash.
3. Agent / skill: put the value in env `ARTIFACT_PUBLISHER_KEY` or the client
   secret store. **Never** ask the user to paste the key (or any
   `ar_pk_…` / `ar_oat_…` / refresh token) into chat.
4. If a key was exposed in chat, logs, or a repo: tell the user to revoke it in
   the desk and mint a new one — do not “fix” by echoing the old value.

## OAuth connect

When the MCP client supports browser consent (production `OAUTH_MODE` on):

1. Register resource `https://app.artifactpublisher.com/mcp`.
2. User completes consent in the browser on the site origin.
3. Tokens stay in the **client** secret store.
4. Never ask the user to paste access tokens, refresh tokens, or auth codes
   into chat.

Headless / always-on agents (Hermes gateway, OpenClaw operators): publisher key
via env remains the practical path.

## What this skill must not do

- Invent billing portal URLs, Stripe Dashboard links, or invoice amounts.
- Print or store secrets in artifacts, commits, or skill files.
- Claim a CLI name (`artifactpublisher` or otherwise) — none is reserved yet.
- Claim `npx skills add ni3do/artifact-publisher` while the product repo is private.
