# Otto AI — Cowork plugin

The client-facing package: skills that carry the operating knowledge, plus the
two MCP servers that carry the data access. See
`../../.claude/plan/distribution-model-review.md` for why the surfaces split
the way they do.

## What's in it

| Component | What it is | Where it runs |
|---|---|---|
| `skills/gohighlevel` | CRM playbooks — leads, conversations, pipeline, the approval-gated write flow | talks to the **hosted** server |
| `skills/limoanywhere` | Back-office playbooks — schedule, quotes, reservations, revenue | talks to the **local** server |
| `commands/otto-setup` | Captures the operator's LimoAnywhere login | local |
| `commands/otto-doctor` | End-to-end diagnostic across both connections | both |
| `otto-limoanywhere` (`.mcp.json`) | Bundled stdio server, read-only | operator's machine |
| `otto-ai-mcp` (hosted, separate) | Remote connector, OAuth via Booked Rides | Railway |

LimoAnywhere runs locally so the operator's login stays on their machine and LA
sees their own IP. GoHighLevel stays hosted so the analysis, the scorecard, and
the write-approval gate remain server-side — where revoking a grant actually
cuts access.

## Before it works

**Set the hosted URL.** `.mcp.json` ships with `SET-YOUR-RAILWAY-DOMAIN-HERE`.
Replace it with the real Railway domain or the GoHighLevel half won't connect.
`npm run plugin:build` warns while the placeholder is still there.

## Building

```bash
npm run plugin:build
```

Compiles the TypeScript, copies `dist/` plus the four runtime dependencies into
`server/`, and verifies the packaged server starts standalone and advertises
its eight LimoAnywhere tools. `server/` is generated and gitignored — an
installed plugin can't run `npm install` for itself, so the payload is
assembled at build time.

## Installing

**From a file** (simplest for testing): build, then in Cowork open
**Customize → Plugins**, choose the upload option, and select this directory.

**From this repository as a marketplace**: `.claude-plugin/marketplace.json` at
the repo root declares it. In Cowork, **Add marketplace** and enter
`Limo-Marketer/otto-ai-plugin-src`. Note this requires `server/` to be present in the
repository, which it currently isn't — see below.

## Open items

- **`server/` is gitignored**, so the marketplace path won't serve a working
  plugin from this repo yet. The intended fix is a separate public marketplace
  repo holding built packages, keeping vendored `node_modules` out of the
  source tree. Until then, use file upload.
- **Private-repo marketplaces** — unconfirmed whether Cowork can authenticate
  to a private GitHub repo. Another reason the marketplace repo may need to be
  public or unlisted.
- **Sandbox runtime** — the bundled server assumes Cowork's VM can run
  `node`. Verified locally, not yet inside Cowork.
- **Network egress** — assumes the sandbox permits outbound HTTPS to
  `manage.mylimobiz.com`. If it doesn't, LimoAnywhere has to stay hosted.

## What goes where

> Put it in a skill if you'd be comfortable posting it publicly. Put it behind
> the hosted connector if you wouldn't.

Skills ship as plaintext to client machines. The scorecard logic and anything
else worth protecting stay on the server behind a tool call.
