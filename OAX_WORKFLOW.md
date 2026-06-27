# oax-tasker — CI auto-fix bot

Push to dev → wait for GitHub Actions → on failure: run `openaxe` to fix → push → repeat until green.

## Quick start

```bash
# Start the loop (blocks until CI passes or max iterations hit)
./oax-auto

# Or, run in background tmux
tmux new-session -d -s oax-bot './oax-auto; tmux wait-for -S oax-bot-done'
# Check back later
tmux capture-pane -pt oax-bot
# When done
tmux kill-session -t oax-bot
```

## Individual steps

```bash
# Push dev, save commit SHA to /tmp/oax-last-sha
./oax-push

# Wait for CI to finish (exits 0 on pass, non-zero on fail)
./oax-wait

# Download last CI failure log (prints log path)
LOGFILE=$(./oax-fix --download-only)

# Run fix session against a saved CI log
./oax-fix "$LOGFILE"
```

## How it works

**`oax-auto`** loops:
1. `oax-push` — `git push origin dev`
2. `oax-wait` — `gh run watch --exit-status --branch dev`
3. On pass → exit 0 🎉
4. On fail → `oax-fix --download-only` saves the CI log
5. `oax-fix <log>` — runs `openaxe run --dangerously-skip-permissions --file <log> "Fix these..."` — **headless mode**, no TUI
6. Checks for new commits → pushes → goes to step 2
7. Reaches `$OAX_MAX_ITER` (default 5) → exits 1

The `openaxe run` command runs in **non-interactive mode** (no TUI) — it sends the prompt, processes, and exits when the session goes idle.

## Config

| Env var | Default | Description |
|---------|---------|-------------|
| `OAX_REPO_DIR` | `/home/dressedinblack/.openaxe/packages/openaxe` | Repo path |
| `OAX_MAX_ITER` | `5` | Max loop iterations |

## Caveats

- CI takes ~30 min per iteration. Budget a few hours.
- Each fix session spawns a fresh `openaxe` session with no memory of past fixes.
- The agent works headlessly — you don't see what it's doing. Check its commit messages.
- `--dangerously-skip-permissions` means the agent can modify any file. This is what you want for CI fixes.
