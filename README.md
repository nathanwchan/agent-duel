# agent-duel

Run Codex and Claude Code in parallel on separate git worktrees, then have each agent review both implementations. You pick the winner and `agent-duel` switches your main repo to that branch.

<figure>
  <img width="1726" height="1010" alt="agent-duel implementing" src="https://github.com/user-attachments/assets/357d0b95-9bb5-4caf-8e6d-7af0d9b09a7a" />
  <figcaption>Codex and Claude Code implement a feature in parallel.</figcaption>
</figure>

  <br />
  <br />
  
<figure>
  <img width="1726" height="1010" alt="agent-duel review+winner" src="https://github.com/user-attachments/assets/b42ec195-4a5c-4bdf-95ff-3673ec40e64c" />
  <figcaption>Both agents compare each others' diffs and then highlight the winner in the right pane; after you make your selection, agent-duel cleans up worktrees, switches the repo to that branch, and exits tmux.</figcaption>
</figure>

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
- ChatGPT subscription (for Codex CLI access)
- Claude subscription (for Claude Code CLI access)
- No OpenAI/Anthropic API usage: this runs the local `codex` and `claude` CLIs in your terminal.

## Install

Option A: add this repo to your PATH:

```bash
export PATH="$PATH:/path/to/agent-duel/bin"
```

Option B: copy scripts to `~/.local/bin`:

```bash
cp bin/* ~/.local/bin/
```

Note: keep all scripts from `bin/` together.

## Upgrade

If you copied to `~/.local/bin`, re-copy after updates:

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
4. Opens a right-side winner pane that waits for both reviews, then clearly shows the result (winner or tie) and prompts you to choose.
5. Selecting a winner removes the worktrees, switches your main repo to the winning branch, prints the latest commit, and closes tmux.

## Lifecycle (core behavior)

1. **Implement phase:** Codex/Claude run in their worktrees until each writes its commit message file and creates a `*.done` file (`codex.done` / `claude.done`). If either agent exits without touching its `*.done` file, the session will wait indefinitely.
2. **Auto-commit phase:** Once both `*.done` files exist, `agent-duel` auto-commits each worktree using the first line of `codex.commit.txt` / `claude.commit.txt` (fallback: `Update code`). If git identity is missing, the commit will fail and the run will stop.
3. **Review phase:** After commits, both agents are started again with a fixed review prompt and write `codex.review.txt` / `claude.review.txt`. A combined `summary.md` is generated. When both reviews arrive, a short chime plays.
4. **Select phase:** The winner is chosen via the built-in selector pane, or later using `agent-duel-select /tmp/agent-duel/<repo>/<feature>` (useful if tmux closes or you want to defer the decision). Selecting a winner switches your main repo to that branch and closes the tmux session.

Commit messages:
- Each agent writes its own suggested commit message to a file in the session directory.
- Both branches are committed (if needed) after implementation and before review.
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
- `consensus.txt`, `selected.txt`, `committed.txt`
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
- `AGENT_DUEL_CODEX_PROMPT_DELAY` (default: 0.5)
- `AGENT_DUEL_CLAUDE_PANE_PERCENT` (default: 50)
- `AGENT_DUEL_CLAUDE_PROMPT_DELAY` (default: 2)
- `AGENT_DUEL_CLAUDE_ENTER_DELAY` (default: 0.3)
- `AGENT_DUEL_CLAUDE_PERMISSION_MODE` (default: `bypassPermissions`)
- `AGENT_DUEL_CLAUDE_ADD_DIR` (default: session dir)
- `AGENT_DUEL_CODEX_APPROVAL_POLICY` (default: `never`)
- `AGENT_DUEL_CODEX_SANDBOX` (default: `workspace-write`)

## Notes / troubleshooting

- If `agent-duel` prints `agent-duel-lib: No such file or directory`, re-copy all scripts from `bin/`.
- If you see old behavior, check `command -v agent-duel`.
- If Claude doesn’t auto-start the prompt, try:
  ```bash
  AGENT_DUEL_CLAUDE_PROMPT_DELAY=4 agent-duel -m "..." --reuse
  ```
- If you see `branch ... is already used by worktree`, switch that worktree back to `main` or `master` (or another branch), then retry.
- No tests are run by default.

## License

MIT
