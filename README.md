# exodus-releases

Release feed for the **exodus** CLI — the Claude-Code-driven operator for Grounding Co and related ad workflows.

## Installing or updating

Paste this into Claude Code, then confirm when it asks:

> Install the latest exodus by following the instructions at https://github.com/bdavis905/exodus-releases/blob/main/INSTALL.md

That one sentence works whether you're setting up for the first time or pulling down an update. Claude Code will detect which scenario you're in and do the right thing.

For the full procedure (in case Claude Code asks or you want to review it yourself), see [INSTALL.md](./INSTALL.md).

## What's shipped per release

Each release has two zip assets:

- **`exodus-update-<version>.zip`** (~100 KB) — overlay for existing installs. Only touches `exodus/` and `.claude/skills/`. Your `.env`, `state/`, `output/`, and saved work are left alone.
- **`grounding-co-fresh-<version>.zip`** (~250 KB) — full workspace scaffold for first-time installs. Excludes `node_modules` (regenerated via `npm install`) and any `.env` file (you create your own from the included `.env.example`).

## Manual / offline fallback

If Claude Code can't reach GitHub, download the appropriate asset from the [latest release](https://github.com/bdavis905/exodus-releases/releases/latest) and follow the instructions in INSTALL.md, substituting the local path for the `curl` download.

## Reporting issues

Ping Brad directly. This repo only hosts releases — it doesn't accept issues or pull requests. The source is in a separate private repo.
