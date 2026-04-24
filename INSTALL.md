# Installing and Updating Exodus via Claude Code

This file is a procedure for **Claude Code** to follow on behalf of a user. If you're a human, you can just paste the following into Claude Code and the rest of this file will execute itself:

> Install the latest exodus by following the instructions at https://github.com/bdavis905/exodus-releases/blob/main/INSTALL.md

The same sentence works for a first-time install and for pulling down updates.

---

## Procedure (for Claude Code)

Follow these steps in order. Do not skip or reorder them. Report each step to the user as you go — they should see what you're doing.

### Step 1 — Detect the install state

Determine whether the user already has an exodus install. Search these paths in order and stop at the first match for a directory containing **both** `exodus/package.json` and `.claude/skills/`:

1. Current working directory (if it's called `grounding-co`)
2. `$HOME/grounding-co`
3. `$HOME/Documents/grounding-co`
4. `$HOME/Desktop/grounding-co`

If found → this is an **UPDATE**. Remember the absolute path to the workspace root (the `grounding-co/` folder itself). Skip to Step 3.

If not found → this is a **FRESH INSTALL**. Continue to Step 2.

### Step 2 — Pick a location (fresh install only)

Ask the user:

> "I don't see an existing exodus install. Where should I install it? I'll suggest `$HOME/Documents/grounding-co` unless you'd rather have it somewhere else."

Wait for their answer. If they accept the default, use `$HOME/Documents/grounding-co`. If they give a different path, use that. The final path is the **workspace root**. Its parent is the **parent directory**.

### Step 3 — Fetch the latest release metadata

Run:

```bash
curl -sL https://api.github.com/repos/bdavis905/exodus-releases/releases/latest
```

Parse the JSON response. You need:
- `tag_name` (for reporting)
- `body` (for release notes to summarize to the user)
- For UPDATE: the asset whose name starts with `exodus-update-` — use its `browser_download_url`
- For FRESH: the asset whose name starts with `grounding-co-fresh-` — use its `browser_download_url`

### Step 4a — Apply the overlay (UPDATE path only)

Tell the user the version you're installing and summarize the release notes briefly. Then run:

```bash
# Where PARENT is the parent directory of the existing grounding-co/ folder
# and URL is the exodus-update-*.zip download URL from step 3.

curl -sL "$URL" -o /tmp/exodus-update.zip
cd "$PARENT"
unzip -o /tmp/exodus-update.zip
rm /tmp/exodus-update.zip

cd "$WORKSPACE_ROOT"
node exodus/dist/bin/exodus.js doctor
```

The overlay only writes to `exodus/` and `.claude/skills/` inside the existing workspace. Nothing else is modified. The user's `.env`, `state/`, `output/`, and `node_modules/` are preserved by construction — they're not in the zip.

After doctor runs, report its result to the user.

### Step 4b — Extract the fresh install (FRESH path only)

Tell the user the version you're installing. Then run:

```bash
# Where PARENT is the parent of the workspace root,
# WORKSPACE_ROOT is the target grounding-co/ path,
# and URL is the grounding-co-fresh-*.zip download URL from step 3.

mkdir -p "$PARENT"
curl -sL "$URL" -o /tmp/grounding-co-fresh.zip
cd "$PARENT"
unzip -o /tmp/grounding-co-fresh.zip
rm /tmp/grounding-co-fresh.zip

cd "$WORKSPACE_ROOT"
npm install
```

Then set up credentials. Show the user the API-key dashboard link:

> "Open https://agent-dash-groundco.vercel.app/settings?tab=api and generate an API key. Paste it here when you have it, and I'll drop it into your `.env` file."

When they paste the key:

```bash
cp "$WORKSPACE_ROOT/.env.example" "$WORKSPACE_ROOT/.env"
# Replace "paste-your-key-here" with the actual key they pasted.
# Use sed or edit the file directly — the placeholder text is literally "paste-your-key-here".
```

Then run doctor to confirm everything works:

```bash
cd "$WORKSPACE_ROOT"
node exodus/dist/bin/exodus.js doctor
```

Report its result.

### Step 5 — Summary

Tell the user:
- What was installed (version + path)
- Whether doctor was green
- If UPDATE: nothing further — they can keep working. Future updates: they can just say "check for updates" or "update exodus" and you'll handle it.
- If FRESH: remind them they can now open `README.md` inside the workspace or visit the dashboard at `https://agent-dash-groundco.vercel.app` to start using the pipelines.

### If anything fails

- **Doctor fails after UPDATE:** the overlay created no backup of its own (Claude Code is driving, not the CLI's built-in updater). Tell the user which check failed, what doctor suggests, and offer to investigate. Do not attempt to roll back unless the user asks.
- **Download fails:** surface the error. Offer to retry. If GitHub is unreachable, ask the user if they have a local zip file they want to use instead.
- **Extraction fails mid-way:** the files may be in a half-applied state. Do not pretend it succeeded. Tell the user exactly what ran and what didn't, and ask how they want to proceed.

---

## What this install contains

- **CLI:** `exodus` — command-line tool for running ad-generation pipelines
- **Skills:** `.md` files in `.claude/skills/` that Claude Code reads to operate each pipeline (spark, viral, mirror, remix, pulse, intel, scout, image-ads, browse, update, creative-strategy, drive)
- **References:** foundational creative-strategy docs the skills draw on
- **Docs:** README, PIPELINES.md, CLAUDE.md, ONBOARDING-CHECKLIST.md
- **State scaffolding:** a brand-profile template the user fills in

No prompts, no server logic, no secrets. The pipelines themselves run on the hosted dashboard; this is the local operator interface.
