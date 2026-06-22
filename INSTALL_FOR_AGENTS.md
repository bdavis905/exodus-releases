# Install Exodus + Genesis (for your coding agent)

You are an AI coding agent setting up Exodus (automatic transmission) and Genesis
(manual transmission) for your user in ONE install. Follow these steps in order.
NEVER ask the user to paste a secret into this chat — they paste it into a file.

## 0. Host check
This guide is verified for **Claude Code**. Other agents (Codex, Cursor) usually
work — adapt paths to your skills directory. If you are not sure which host you
are, ask the user.

## 1. Pick the install folder
Create or use a parent folder named `Exodus`. All of the user's brands live as
subfolders inside it. `cd` into it.

## 2. Idempotency check (safe re-run)
If `./.env` already contains `EXODUS_API_KEY=` AND `GENESIS_API_KEY=`, the install
is already provisioned — skip to step 6 (verify). Do not re-mint keys.

## 3. Install the Exodus CLI + skills
Download the latest Exodus fresh-install zip from the releases page
(github.com/bdavis905/exodus-releases — see its INSTALL.md for the current asset
URL) and unpack it here. This lays down the `exodus` CLI and the `exodus-*` skills
under `.claude/skills/`.

## 4. Lay down the Genesis manual-transmission skill
Download https://gas.copycoders.ai/downloads/genesis-bots-skill.zip and unzip it
into `.claude/skills/` so the skill lands at `.claude/skills/genesis-bots`. Delete
the zip afterward.

## 5. Create the single root .env, then STOP for the paste
If `./.env` does not exist, create it with this scaffold (comments only — no
secrets), and ensure `.env` is covered by `.gitignore`:

    # Exodus + Genesis config — paste your dashboard .env block below this line.
    # Get it from your Exodus dashboard → Settings → Claude Code → "Copy .env block".

Then STOP and tell the user: "Open your Exodus dashboard → Settings → Claude Code,
click **Copy .env block**, paste it into `<folder>/.env` below the comment, save,
and tell me when done." Do not ask them to paste it into this chat.

## 6. Verify
After the user confirms:
- Run `node exodus/dist/bin/exodus.js doctor` — expect all green.
- Confirm the Genesis manual path: list the available bots with the gen_ key
  (read `.claude/skills/genesis-bots/SKILL.md` for the exact list call). Expect a
  non-empty bot list.
Report both results to the user.

## 7. Done
Remind the user that skills load at session start — open a new session (or /clear)
before running pipelines. For Exodus image generation and bot LLM usage, add the
Anthropic/OpenRouter and KIE keys in the dashboard (not the .env); Exodus syncs the
provider key down for the manual path.
