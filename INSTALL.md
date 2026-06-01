# Installing and Updating Exodus via Claude Code

**Exodus** is a command-line tool for running ad-generation pipelines. This page is its install guide. You install it into a project folder on your own computer; the pipelines themselves run on the hosted Exodus dashboard (the one you signed up for) — this is just the local operator interface.

If you use **Claude Code** (the Desktop app's Code tab, or the CLI), it can carry out the steps below for you. Generate your personalized install snippet from **Settings → Claude Code** on the dashboard — the **Copy install instructions** button builds a snippet that already includes your API key and brand. Paste that into Claude Code. The same snippet works for a first-time install and for pulling down updates.

**What gets set up, and why it's safe to approve:**

- The Exodus workspace is downloaded into a new folder you choose, and `npm install` runs there.
- Your API key is written into a local `.env` file **inside that folder** so the CLI can reach your dashboard. The key stays on your machine.
- The pipeline **skills** (`.md` files) are added to that folder's `.claude/` directory. These are **project-local**: they only apply when you work inside this folder, and they do **not** change your global Claude Code setup or how Claude behaves in any other project.

Claude Code may ask your permission for some of these steps (writing the `.env`, adding the skill files). That's expected — approving them is what installs the tool.

---

## Steps for Claude Code

Work through these in order. Don't skip or reorder them. Report each step to the user as you go.

**Adapt commands to the user's operating system.** The shell snippets below are written for **macOS / Linux** (bash/zsh). If the user is on **Windows**, run the PowerShell equivalent — do not paste bash verbatim into PowerShell. Common translations:

| Task | macOS / Linux (bash) | Windows (PowerShell) |
|------|----------------------|----------------------|
| Home directory | `$HOME` | `$env:USERPROFILE` |
| Download a file | `curl -sL <url> -o <path>` | `Invoke-WebRequest <url> -OutFile <path>` (alias `iwr`) |
| Fetch JSON | `curl -sL <url>` | `Invoke-RestMethod <url>` (alias `irm`) |
| Unzip | `unzip -o file.zip` | `Expand-Archive -Force file.zip -DestinationPath .` |
| Move / rename | `mv a b` | `Move-Item a b` |
| Copy | `cp a b` | `Copy-Item a b` |
| Temp dir | `/tmp` | `$env:TEMP` |
| Path separator | `/` | `\` (PowerShell also accepts `/`) |

`node`, `npm`, and `npx` are identical on every OS — run those as written. **Run the Exodus CLI as `node exodus/dist/bin/exodus.js <command>` during install and update**, not `npx exodus <command>`. The `node` form runs the already-installed local build directly; some sandboxed environments block `npx` for a package they don't yet recognize, even when it's already on disk. When in doubt, prefer your own file read/write/edit tools over shell text-munging (`sed`, `for` loops): editing a file directly is OS-agnostic and less error-prone.

### Step 0 — Read brand info from the user's message

The dashboard's "Copy install instructions" button produces a paste sentence containing:

> `BRAND_NAME=<DisplayName>, BRAND_SLUG=<slug>`

If the user's message contains `BRAND_NAME=...` and `BRAND_SLUG=...`, capture both values. They drive the workspace folder name and brand-token substitution in Step 4b.

If the user's message has no `BRAND_NAME`/`BRAND_SLUG` (older paste, or manual install), ask:

> "Which brand is this install for? I need the display name (e.g., 'Flow') and the slug (e.g., 'flow') the dashboard uses for it."

Wait for both before continuing.

### Step 1 — Detect the install state

Determine whether the user already has an exodus install. Search in this order, stopping at the first match for a directory containing `exodus/package.json`:

1. The user's current working directory (most reliable signal — they cd'd here for a reason)
2. `$HOME/Documents/<BRAND_SLUG>` — the canonical fresh-install location
3. `$HOME/<BRAND_SLUG>`
4. `$HOME/Desktop/<BRAND_SLUG>`
5. Legacy paths from older installs: `$HOME/Documents/grounding-co`, `$HOME/grounding-co`, `$HOME/Desktop/grounding-co`

If found → this is an **UPDATE**. Remember the absolute path as the **workspace root**. Skip to Step 3.

If none match → this is a **FRESH INSTALL**. Continue to Step 2.

### Step 2 — Pick a location (fresh install only)

Suggest the canonical default and let the user override:

> "I don't see an existing exodus install for `<BRAND_SLUG>`. I'll install it at `$HOME/Documents/<BRAND_SLUG>` unless you'd rather have it somewhere else."

Whatever path they accept becomes the **workspace root**. Its parent is the **parent directory**.

### Step 2.5 — Ensure Node.js 20+ is installed

Everything from here on needs **Node.js 20 or newer** (`npm`, `npx`, and the CLI all depend on it). The Claude Desktop app does not bundle Node, so check for it:

```bash
node --version
```

If `node` is missing, or the major version is below 20, install it and then continue:

- **macOS:** `brew install node` if Homebrew is present; otherwise download the LTS installer from https://nodejs.org and run it.
- **Windows:** `winget install OpenJS.NodeJS.LTS` if winget is present; otherwise download the LTS installer from https://nodejs.org and run it.
- **Linux:** use the distro package manager, or https://nodejs.org.

After installing, confirm `node --version` reports v20+ before moving on. If a just-installed Node isn't picked up, the user may need to open a fresh terminal (or restart the Code session). Report what you installed.

### Step 3 — Fetch the latest release metadata

Run:

```bash
curl -sL https://api.github.com/repos/bdavis905/exodus-releases/releases/latest
```

Parse the JSON. You need:
- `tag_name` (for reporting)
- `body` (for release-notes summary)
- For UPDATE: the CLI handles asset selection — you only need `tag_name` and `body` for reporting.
- For FRESH: the asset whose name starts with `exodus-workspace-` — use its `browser_download_url`. If only `grounding-co-fresh-*` is available (release published before May 8 2026), fall back to that and skip Step 4b's token-substitution sub-step (the legacy zip has no placeholders).

### Step 4a — Apply the overlay (UPDATE path)

Delegate to the CLI's built-in updater — it handles version checking, backup, extraction, and rollback in one shot:

```bash
cd "$WORKSPACE_ROOT"
node exodus/dist/bin/exodus.js update --force
node exodus/dist/bin/exodus.js doctor
```

`update` writes only to `exodus/` and `.claude/skills/`. The user's `.env`, `state/`, `output/`, and `node_modules/` are preserved by construction. Backups go to `.backup/<timestamp>/` so the user can roll back with `node exodus/dist/bin/exodus.js update --rollback` if doctor regresses.

If the CLI itself won't run (e.g., the workspace is corrupted), fall through to the raw extraction:

```bash
# Manual fallback only — prefer the CLI updater above.
curl -sL "$URL" -o /tmp/exodus-update.zip
cd "$WORKSPACE_ROOT/.."
unzip -o /tmp/exodus-update.zip
rm /tmp/exodus-update.zip
cd "$WORKSPACE_ROOT"
node exodus/dist/bin/exodus.js doctor
```

Note: the overlay zip's top-level folder is `grounding-co/` because the CLI's update command hardcodes that name. For a workspace folder that the customer renamed (e.g., `flow/`), the manual fallback creates a stray `grounding-co/` next to the workspace — move its `exodus/` and `.claude/` contents into the real workspace and remove the stray. This is rare; the CLI updater is the supported path.

After doctor runs, report its result to the user.

### Step 4b — Extract the fresh install (FRESH path)

Tell the user the version. Then run, using `BRAND_SLUG` from Step 0:

```bash
# PARENT is the parent of WORKSPACE_ROOT.
# URL is the exodus-workspace-*.zip download URL from Step 3.

mkdir -p "$PARENT"
curl -sL "$URL" -o /tmp/exodus-workspace.zip
cd "$PARENT"
unzip -o /tmp/exodus-workspace.zip
rm /tmp/exodus-workspace.zip
```

The zip extracts a folder called `_workspace/`. Rename it to the customer's brand slug:

```bash
mv "$PARENT/_workspace" "$WORKSPACE_ROOT"
```

**Substitute brand tokens.** The customer-facing docs inside the zip carry `{{BRAND_NAME}}` and `{{BRAND_SLUG}}` placeholders that must be resolved before the workspace is usable. Use `BRAND_NAME` and `BRAND_SLUG` from Step 0 (they are literal strings — no shell expansion or quoting tricks).

```bash
cd "$WORKSPACE_ROOT"
for f in README.md PIPELINES.md .claude/skills/scout.md; do
  if [[ -f "$f" ]]; then
    sed -i.bak \
      -e "s|{{BRAND_NAME}}|<BRAND_NAME literal>|g" \
      -e "s|{{BRAND_SLUG}}|<BRAND_SLUG literal>|g" \
      "$f" && rm -f "$f.bak"
  fi
done
```

Replace `<BRAND_NAME literal>` and `<BRAND_SLUG literal>` with the actual captured values. If a value contains a `|` character (extremely unlikely for a brand name, impossible for a slug), use a different sed delimiter.

> **Simpler + OS-agnostic:** instead of the `sed` loop, you may just open each of those files and replace the `{{BRAND_NAME}}` / `{{BRAND_SLUG}}` placeholders with your own edit tool. On Windows this is the preferred path — there's no system `sed`.

**Install dependencies:**

```bash
npm install
```

**Set up credentials.**

Look in the user's original message for the two config lines the dashboard's
"Copy install instructions" button includes:

> `EXODUS_API_URL=https://<backend>.convex.site`
> `EXODUS_API_KEY=vad_XyZ123...`

If both are present → use them. If the URL is missing (older paste / manual
install) → ask which environment they signed up on and use:
- **xo.copycoders.ai** (customers) → `https://accomplished-tapir-106.convex.site`
- **dev.xo.copycoders.ai** (internal team) → `https://good-cod-360.convex.site`

If the key is missing → ask:

> "I'll need an API key to finish setup. On your dashboard open **Settings → Claude Code** and click **Copy install instructions** — that generates a key and the matching backend URL."

Then write `.env`. **Copy `.env.example` to `.env` and set BOTH lines** —
`EXODUS_API_URL` (replacing the `https://YOUR-EXODUS-BACKEND.convex.site`
placeholder) and `EXODUS_API_KEY` (replacing `paste-your-key-here`):

```bash
cp "$WORKSPACE_ROOT/.env.example" "$WORKSPACE_ROOT/.env"
sed -i.bak \
  -e "s|https://YOUR-EXODUS-BACKEND.convex.site|REAL_API_URL_HERE|" \
  -e "s|paste-your-key-here|REAL_KEY_HERE|" \
  "$WORKSPACE_ROOT/.env"
rm -f "$WORKSPACE_ROOT/.env.bak"
```

> **Simpler + OS-agnostic (preferred on Windows):** copy `.env.example` to `.env`
> and edit the `EXODUS_API_URL=` and `EXODUS_API_KEY=` lines directly with your
> own tool. Getting `EXODUS_API_URL` right is what points the CLI at the correct
> backend — a mismatch is the #1 cause of a 401 from `whoami`.

**Run doctor:**

```bash
cd "$WORKSPACE_ROOT"
node exodus/dist/bin/exodus.js doctor
```

Report its result.

### Step 5 — Finish: start a fresh session so the skills load

The pipeline skills were just written into the workspace's `.claude/` folder, and Claude Code only loads skills **when a session starts**. They are therefore **not active in this current session** — the user must open a new one.

Tell the user:
- What was installed (version + path) and whether doctor was green.
- **Do this next:** open a **new Claude Code session in the workspace folder** (`<WORKSPACE_ROOT>`, e.g. `~/Documents/flow`) so the pipeline skills load. Until they restart there, the pipelines won't be available.
- Once they're in that fresh session, they can just say what they want in plain English (e.g. "I want to write some ads about …") to start, or open `README.md` in the folder. Your dashboard is at https://xo.copycoders.ai (or https://dev.xo.copycoders.ai for the internal team).
- For UPDATE: same — restart the session in the workspace folder so the refreshed skills load. Future updates: they can say "check for updates" or "update exodus" anytime.

### If anything fails

- **Doctor fails after UPDATE:** the overlay creates no backup of its own. Tell the user which check failed, what doctor suggests, and offer to investigate. Do not roll back unless asked.
- **Token substitution missed something:** if the user opens a doc and sees `{{BRAND_NAME}}` or `{{BRAND_SLUG}}` somewhere, run the sed loop again with the correct values. Templating is safe to re-apply.
- **Download fails:** surface the error. Offer to retry. If GitHub is unreachable, ask the user if they have a local zip file to use.
- **Extraction fails mid-way:** the files may be in a half-applied state. Do not pretend it succeeded. Tell the user what ran and what didn't, and ask how they want to proceed.

---

## What this install contains

- **CLI:** `exodus` — command-line tool for running ad-generation pipelines
- **Skills:** `.md` files in `.claude/skills/` that Claude Code reads to operate each pipeline (spark, viral, mirror, remix, pulse, intel, scout, image-ads, images, template, meme, creative, pixar, swipe, browse, update, creative-strategy, strategist-mode, drive, foundation, genesis, and the primer-clone variants)
- **References:** foundational creative-strategy docs the skills draw on
- **Docs:** README, PIPELINES.md, CLAUDE.md, ONBOARDING-CHECKLIST.md (templated to the customer's brand at install time)
- **State scaffolding:** a brand-profile template the customer fills in

No prompts, no server logic, no secrets. The pipelines themselves run on the hosted dashboard; this is the local operator interface.
