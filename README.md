# agent-duel

Run Codex and Claude Code in parallel on separate git worktrees, then compare their implementations and pick a winner.

## Requirements
- git
- tmux
- codex CLI
- claude CLI

Optional (for prettier review rendering in tmux panes):
- glow (recommended)
- mdcat
- bat

## Install

Option A: add this repo to your PATH:

```bash
export PATH="$PATH:/path/to/agent-duel/bin"
```

Option B: copy scripts to ~/.local/bin:

```bash
cp bin/* ~/.local/bin/
```

## Usage

From inside a git repo:

```bash
agent-duel -m "your feature description"
```

Or pass a prompt file:

```bash
agent-duel -f /path/to/prompt.txt -n feature-name
```

The flow:
1. Creates /tmp worktrees for `feat/codex` and `feat/claude`.
2. Launches Codex and Claude in tmux panes to implement in parallel.
3. Launches review prompts for both.
4. Shows Codex review (left), Claude review (right), and a winner prompt (bottom).
5. Selecting a winner switches your main repo to the winning branch and closes tmux.

## Environment variables
- `AGENT_DUEL_STARTUP_DELAY` (default: 1) - tmux pane startup delay
- `AGENT_DUEL_CLAUDE_PROMPT_DELAY` (default: 2) - delay before sending prompt to Claude
- `AGENT_DUEL_CLAUDE_PERMISSION_MODE` (default: bypassPermissions)
- `AGENT_DUEL_CLAUDE_ADD_DIR` (default: session dir)
- `AGENT_DUEL_CODEX_APPROVAL_POLICY` (default: never)
- `AGENT_DUEL_CODEX_SANDBOX` (default: workspace-write)

## Notes
- The scripts use `/tmp/agent-duel` (and `/private/tmp/agent-duel` on macOS).
- Old /tmp worktrees are cleaned on the next run.
- The winner is auto-committed if there are uncommitted changes in the worktree.
- No tests are run by default.

## License
MIT
