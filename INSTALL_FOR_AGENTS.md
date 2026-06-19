<!-- dev-branch draft of the agent install guide. Tested from the dev dashboard; promote dev → main to publish for customers. No secrets ever. -->
# Install Exodus + Genesis (for your coding agent)

You are an AI coding agent setting up Exodus (automatic transmission) and Genesis
(manual transmission) for your user in ONE install. Follow these steps in order and
report each step as you go. **NEVER ask the user to paste a secret into this chat —
keys go into a file the user pastes themselves.**

Commands below are written for **macOS / Linux** (bash/zsh). On **Windows**, use the
PowerShell equivalents (`iwr`/`irm` for download, `Expand-Archive` for unzip);
`node`, `npm`, and `npx` are identical on every OS. Prefer your own file read/write
tools over shell text-munging where you can — it's OS-agnostic and less error-prone.

## 0. Host check
This guide is verified for **Claude Code**. Other agents (Codex, Cursor) usually
work — adapt paths to your skills directory. If unsure which host you are, ask.

## 1. Use the folder the user already opened
Install **directly into the folder the user opened** (the current working directory) —
do **NOT** create a separate parent folder. This should be a fresh, empty folder
dedicated to Exodus; it becomes the workspace root, and the user's brands live as
subfolders inside it. If the opened folder already has unrelated work in it, ask the
user to open a new empty folder for Exodus instead.

## 2. Capture brand info
The dashboard's install snippet includes a line like
`BRAND_NAME=<DisplayName>, BRAND_SLUG=<slug>`. Capture both values — they drive the
brand-token substitution in step 5 and the `brand use` call in step 8. If they're
missing (older paste), ask the user for the brand display name and slug.

## 3. Idempotency check (safe re-run)
If `./.env` already contains `EXODUS_API_KEY=` AND `GENESIS_API_KEY=`, the install is
already provisioned — skip to step 6. Do not re-mint or overwrite keys.

## 4. Ensure Node.js 20+
Run `node --version`. If `node` is missing or below v20, install the LTS
(macOS: `brew install node` or nodejs.org; Windows: `winget install OpenJS.NodeJS.LTS`
or nodejs.org; Linux: distro package manager) and confirm `node --version` is v20+
before continuing. A freshly installed Node may need a new terminal/session.

## 5. Install the Exodus CLI + skills (into this folder)
Fetch the latest release metadata:

```bash
curl -sL https://api.github.com/repos/bdavis905/exodus-releases/releases/latest
```

From the JSON, take `tag_name` (report it) and the asset whose name starts with
`exodus-workspace-` — use its `browser_download_url` as `$URL`. Download and unpack
it **into this folder**:

```bash
curl -sL "$URL" -o /tmp/exodus-workspace.zip
unzip -o /tmp/exodus-workspace.zip -d /tmp/exodus-unpack
rm /tmp/exodus-workspace.zip
```

The zip contains a single top-level `_workspace/` folder. **Move its entire contents
(including dotfiles like `.claude/` and `.env.example`) into the current folder**, then
remove the leftover `_workspace/`. (Use your file tools; on bash:
`mv /tmp/exodus-unpack/_workspace/* /tmp/exodus-unpack/_workspace/.[!.]* . && rm -rf /tmp/exodus-unpack`.)

**Substitute brand tokens** using the captured `BRAND_NAME` / `BRAND_SLUG` (literal
strings). In `README.md`, `PIPELINES.md`, and `.claude/skills/scout.md`, replace every
`{{BRAND_NAME}}` and `{{BRAND_SLUG}}` placeholder. Editing the files directly with your
own tool is the simplest, OS-agnostic way.

**Install dependencies:**

```bash
npm install
```

## 6. Lay down the Genesis manual-transmission skill
Download `https://gas.copycoders.ai/downloads/genesis-bots-skill.zip` and unzip it into
`.claude/skills/` so the skill lands at `.claude/skills/genesis-bots`. Delete the zip.

## 7. Create the root .env, then STOP for the paste
Exodus reads the user's API keys from a `.env` at the root of this folder.
- If `./.env` does **not** exist, create it with this scaffold (comments only, no
  secrets) and make sure `.env` is in `.gitignore`:

      # Exodus + Genesis config — paste your dashboard .env block below this line.
      # Get it from your Exodus dashboard → Settings → Claude Code → "Copy .env block".

- If `./.env` **already exists**, leave it as-is (do not overwrite it).

Then **STOP** and tell the user:
> "Open your Exodus dashboard → **Settings → Claude Code**, click **Copy .env block**,
> and paste it into `<folder>/.env` (below the comment if the file is new; just append
> the lines if it already has other keys). Save it and tell me when done."

You do **not** handle the keys yourself. Do not ask the user to paste them into this chat.

## 8. After the user confirms the paste — verify
Now that the keys are in `.env`, run (substituting `BRAND_SLUG`):

```bash
node exodus/dist/bin/exodus.js brand use <BRAND_SLUG>
node exodus/dist/bin/exodus.js update
node exodus/dist/bin/exodus.js whoami
node exodus/dist/bin/exodus.js doctor
```

- `brand use` creates the brand's subfolder and pulls its profile.
- `update` gives every other brand the user owns its own folder.
- `whoami` should resolve to the right brand with the foundation **READY**.
- `doctor` should be all green.

Then confirm the Genesis manual path: list the available bots with the `gen_` key
(read `.claude/skills/genesis-bots/SKILL.md` for the exact list call) — expect a
non-empty bot list. Report all results to the user.

## 9. (Optional, last) Max's skill pack
Ask: "Were you in Max's workshop and want his extra skills?" Only if yes, follow his
pack's install prompt. It coexists with Exodus + Genesis (separate skill namespaces).

## 10. Done
Skills load at **session start** — tell the user to restart Claude Code in this same
folder (or `/clear`) before running pipelines. For Exodus image generation and bot LLM
usage, add the Anthropic/OpenRouter and KIE keys **in the dashboard** (not the `.env`);
Exodus syncs the provider key down for the manual path. The dashboard is at
https://xo.copycoders.ai (or https://dev.xo.copycoders.ai for the internal team).
