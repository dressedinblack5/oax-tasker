# oax-tasker

CI auto-fix bot. Push → wait for CI → on failure, auto-fix → push → repeat until green.

## Requirements

- **`gh`** — [GitHub CLI](https://cli.github.com/), authenticated to the repo
- **`git`** — push, fetch, log
- **`openaxe`** or any AI coding assistant — runs the fix sessions

## Quick start

```bash
# Get the bot
git clone https://github.com/dressedinblack5/oax-tasker ~/oax-tasker

# Use it in any repo
cd ~/Projects/your-repo
~/oax-tasker/oax-auto
```

## Scripts

| Script | What it does |
|--------|-------------|
| `oax-auto` | Main loop: push → wait → fix → push → repeat until CI passes |
| `oax-push` | Push HEAD to dev, record commit SHA |
| `oax-wait` | Watch latest CI run, exit 0 on pass, non-zero on fail |
| `oax-fix --download-only` | Download latest CI failure log, print file path |
| `oax-fix <logfile>` | Run AI fix session on a CI log |

All scripts accept `--help`.

## For AI assistants

Each script is a single-purpose CLI command. From any AI coding assistant:

```bash
# Push latest changes
oax-push

# Wait for CI (blocking)
oax-wait

# Download the failure log
LOGFILE=$(oax-fix --download-only)

# Fix the failures
oax-fix "$LOGFILE"
```

## Configuration

| Env var | Default | Description |
|---------|---------|-------------|
| `OAX_REPO_DIR` | current dir | Path to the git repo |

```bash
OAX_REPO_DIR=/path/to/project oax-auto
```
