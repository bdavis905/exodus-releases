# exodus-releases

Release feed for the [exodus](https://github.com/bdavis905/agent-dash-groundco) CLI.

Each release tagged `exodus-vYYYY.M.D` ships a small overlay zip that updates the CLI and Claude Code skill files inside a client's `grounding-co/` workspace. The overlay never touches `.env`, `state/`, `output/`, or `node_modules`.

## Applying an update

From inside your `grounding-co/` workspace:

```bash
npx exodus update
```

Or ask Claude Code to "check for updates" / "update exodus."

## Offline / air-gapped

If the update host is unreachable, download the release zip manually and apply it locally:

```bash
npx exodus update --file ./exodus-update.zip
```

## Rolling back

Every update creates a timestamped backup under `grounding-co/.backup/`. To restore the most recent:

```bash
npx exodus update --rollback
```
