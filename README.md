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

# Start the auto-fix loop in your project
cd ~/Projects/your-repo
~/oax-tasker/oax-auto
```

When you run `oax-auto`, the bot:

1. **Pushes** your latest commits to `dev` branch — this triggers CI
2. **Waits** for the `test` workflow to finish (blocks until done)
3. If CI **passes** → exits with success
4. If CI **fails** → downloads the failure log, runs `openaxe` to auto-fix the tests, commits the fixes, pushes again, and goes back to step 2
5. Repeats up to 5 cycles, then exits regardless

Your repo needs:
- A `dev` branch with CI configured (GitHub Actions)
- A workflow named `test` (the default; edit `oax-wait` and `oax-fix` for a different name)
- An AI assistant on `PATH` (default: `openaxe`)

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
