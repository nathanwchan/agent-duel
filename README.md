# agent-duel

Run Codex and Claude Code in parallel on separate git worktrees, then have each agent review both implementations. You pick the winner and `agent-duel` switches your main repo to that branch.

No OpenAI/Anthropic API usage: this runs the local `codex` and `claude` CLIs in your terminal.

## Requirements

Core:
- macOS or Linux with a terminal emulator (iTerm2, Terminal.app, etc.)
- `git` with worktree support and identity configured:
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "you@example.com"
  ```
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

Or pass the description as a positional argument:

```bash
agent-duel "your feature description"
```

Or run with no arguments to get an interactive prompt:

```bash
agent-duel
```

Or pass a prompt file:

```bash
agent-duel -f /path/to/prompt.txt -n feature-name
```

What happens:
1. Creates `/tmp/agent-duel/<repo>/<feature>/` with two worktrees.
2. Launches Codex and Claude in tmux panes to implement in parallel.
3. Launches review prompts for both.
4. Keeps the review panes open and adds a bottom prompt to pick a winner (with `***` if both reviewers agree).
5. Selecting a winner commits both branches (if needed), removes the worktrees, switches your main repo to the winning branch, prints the latest commit, and closes tmux.

Commit messages:
- Each agent writes its own suggested commit message to a file in the session directory.
- Both branches are committed (if needed) with their respective messages before you pick a winner.
- If missing, it falls back to `Update code`.

## Branch and worktree behavior

- Base branch is auto-detected: `main` or `master`.
- Creates branches: `agent-duel/codex` and `agent-duel/claude`.
- Uses git worktrees under `/tmp/agent-duel` (or `/private/tmp/agent-duel` on macOS).
- Old `/tmp` worktrees are cleaned on the next run.
- If branches already exist, they are reused automatically.
- By default, the branches are reset to the base branch each run. Use `--no-reset` to keep their current state.
- If a branch is checked out in a non-`/tmp` worktree, you must switch that worktree to a different branch before running.

## Outputs

Session files live in:
```
/tmp/agent-duel/<repo>/<feature>/
```

Includes:
- `codex.review.txt`, `claude.review.txt`
- `summary.md` (combined view)
- `codex.commit.txt`, `claude.commit.txt`
- `consensus.txt`, `selected.txt`
- prompt and status files

## Permissions and safety

Defaults:
- Codex runs with `-a never -s workspace-write`.
- Claude runs with `--permission-mode bypassPermissions` and `--add-dir` set to the session directory.

You can override these with environment variables (see below).

## Security considerations

- **Shared systems**: Session files in `/tmp/agent-duel/` are created with owner-only permissions (umask 077), but verify this meets your security requirements on shared systems.
- **Path disclosure**: Prompts sent to Codex and Claude include absolute filesystem paths, which may reveal your username, project names, and directory structure to OpenAI/Anthropic.
- **Sensitive data**: Avoid including secrets (API keys, passwords) in your feature descriptions, as they are saved to `/tmp` and sent to AI services.

## Environment variables

- `AGENT_DUEL_STARTUP_DELAY` (default: 1)
- `AGENT_DUEL_CODEX_PROMPT_DELAY` (default: 1)
- `AGENT_DUEL_CODEX_ENTER_DELAY` (default: 0.3)
- `AGENT_DUEL_CLAUDE_PROMPT_DELAY` (default: 2)
- `AGENT_DUEL_CLAUDE_ENTER_DELAY` (default: 0.3)
- `AGENT_DUEL_CLAUDE_PERMISSION_MODE` (default: `bypassPermissions`)
- `AGENT_DUEL_CLAUDE_ADD_DIR` (default: session dir)
- `AGENT_DUEL_CODEX_APPROVAL_POLICY` (default: `never`)
- `AGENT_DUEL_CODEX_SANDBOX` (default: `workspace-write`)

## Notes / troubleshooting

- If Claude doesn’t auto-start the prompt, try:
  ```bash
  AGENT_DUEL_CLAUDE_PROMPT_DELAY=4 agent-duel -m "..." --reuse
  ```
- No tests are run by default.

## License

MIT
