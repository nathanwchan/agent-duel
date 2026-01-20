# agent-duel

Run Codex and Claude Code in parallel on separate git worktrees, then have each agent review both implementations. You pick the winner and `agent-duel` switches your main repo to that branch.

No OpenAI/Anthropic API usage: this runs the local `codex` and `claude` CLIs in your terminal.

## Requirements

Core:
- macOS or Linux with a terminal emulator (iTerm2, Terminal.app, etc.)
- `git` with worktree support
- `tmux`
- Codex CLI installed and logged in (`codex` on PATH)
- Claude Code CLI installed and logged in (`claude` on PATH)

Subscriptions:
- ChatGPT Plus (for Codex CLI access)
- Claude Code subscription

Optional (prettier review rendering in tmux panes):
- `glow` (recommended)
- `mdcat`
- `bat`

## Install

Option A: add this repo to your PATH:

```bash
export PATH="$PATH:/path/to/agent-duel/bin"
```

Option B: copy scripts to `~/.local/bin`:

```bash
cp bin/* ~/.local/bin/
```

## Quick start

From inside a git repo:

```bash
agent-duel -m "your feature description"
```

Or pass a prompt file:

```bash
agent-duel -f /path/to/prompt.txt -n feature-name
```

What happens:
1. Creates `/tmp/agent-duel/<repo>/<feature>/` with two worktrees.
2. Launches Codex and Claude in tmux panes to implement in parallel.
3. Launches review prompts for both.
4. Keeps the review panes open and adds a bottom prompt to pick a winner.
5. Selecting a winner commits (if needed), removes the worktrees, switches your main repo to the winning branch, and closes tmux.

## Branch and worktree behavior

- Base branch is auto-detected: `main` or `master`.
- Creates branches: `feat/codex` and `feat/claude`.
- Uses git worktrees under `/tmp/agent-duel` (or `/private/tmp/agent-duel` on macOS).
- Old `/tmp` worktrees are cleaned on the next run.
- If branches already exist, use `--reuse`.

## Outputs

Session files live in:
```
/tmp/agent-duel/<repo>/<feature>/
```

Includes:
- `codex.review.txt`, `claude.review.txt`
- `summary.md` (combined view)
- prompt and status files

## Permissions and safety

Defaults:
- Codex runs with `-a never -s workspace-write`.
- Claude runs with `--permission-mode bypassPermissions` and `--add-dir` set to the session directory.

You can override these with environment variables (see below).

## Environment variables

- `AGENT_DUEL_STARTUP_DELAY` (default: 1)
- `AGENT_DUEL_CLAUDE_PROMPT_DELAY` (default: 2)
- `AGENT_DUEL_CLAUDE_PERMISSION_MODE` (default: `bypassPermissions`)
- `AGENT_DUEL_CLAUDE_ADD_DIR` (default: session dir)
- `AGENT_DUEL_CODEX_APPROVAL_POLICY` (default: `never`)
- `AGENT_DUEL_CODEX_SANDBOX` (default: `workspace-write`)

## Notes / troubleshooting

- If Claude doesn’t auto-start the prompt, try:
  ```bash
  AGENT_DUEL_CLAUDE_PROMPT_DELAY=4 agent-duel -m "..." --reuse
  ```
- If the winner commit fails, set git identity:
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "you@example.com"
  ```
- No tests are run by default.

## License

MIT
