# oax-tasker

CI auto-fix bot. Push → wait → CI fails → auto-fix → push → repeat until green.

Works with **any** AI coding assistant: OpenCode, Claude Code, Codex, Cline, etc.

## Quick start

```bash
git clone https://github.com/dressedinblack5/oax-tasker ~/oax-tasker
cd ~/Projects/your-repo
~/oax-tasker/oax-auto
```

That's it. The bot handles the rest.

## How it works

```
         push ──→ GitHub Actions ──→ pass ──→ ✅ done
          ↑                              │
          │                              ▼
          │                            fail
          │                              │
          └──── fix & commit ←── AI reads CI log
```

1. **Push** your latest commits to the current branch → triggers CI
2. **Wait** for the GitHub Actions workflow to finish
3. **Pass** → exit 0
4. **Fail** → download the CI log, run your AI assistant to fix the tests and commit, push again, go to step 2
5. Repeats up to 5 cycles, then exits

## Scripts

| Script | Action |
|--------|--------|
| `oax-auto` | Full auto-fix loop (push → wait → fix → repeat) |
| `oax-push` | Push HEAD to remote branch, record SHA |
| `oax-wait` | Watch the latest CI run, exit 0 on pass |
| `oax-fix --download-only` | Download latest CI failure log, print path |
| `oax-fix <logfile>` | Run AI fix session on a CI log |

All scripts accept `--help`.

## Configuration

| Env var | Default | Description |
|---------|---------|-------------|
| `OAX_REPO_DIR` | current dir | Path to the git repo |
| `OAX_BRANCH` | current branch | Target branch for push & CI watch |
| `OAX_WORKFLOW` | `test` | GitHub Actions workflow name |
| `OAX_AI_CMD` | `openaxe run ...` | AI assistant command with `{logfile}`, `{repodir}`, `{promptfile}` placeholders |

```bash
# Override repo directory
OAX_REPO_DIR=/path/to/project oax-auto

# Different branch and workflow
OAX_BRANCH=main OAX_WORKFLOW=ci oax-auto
```

## AI assistants

Set `OAX_AI_CMD` to use any AI coding assistant. Placeholders are substituted at runtime:

- `{logfile}` — path to the CI failure log
- `{repodir}` — path to the git repo
- `{promptfile}` — path to a temp file with fix instructions

```bash
# openaxe (default — no config needed)
oax-auto

# Claude Code
OAX_AI_CMD='claude --allowedTools "Bash,Write,Read" -p "$(cat {promptfile})"' oax-auto

# Codex CLI
OAX_AI_CMD='codex "$(cat {promptfile})"' oax-auto

# Cline
OAX_AI_CMD='cline --prompt "$(cat {promptfile})"' oax-auto
```

## Manual use from AI assistants

Each script is a standalone CLI command so AI assistants can use them directly:

```bash
# Push latest code
oax-push

# Wait for CI (blocks until done)
oax-wait

# Download the failure log
LOGFILE=$(oax-fix --download-only)

# Fix the failures
oax-fix "$LOGFILE"
```

## Requirements

- **`gh`** — [GitHub CLI](https://cli.github.com/), authenticated to your repo
- **`git`** — push, fetch, log
- An AI assistant on `PATH` — openaxe, claude, codex, or any CLI AI tool
