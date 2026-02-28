# Install

## Prerequisites

Install these first if you don't have them:

- **Claude Code** — [docs.anthropic.com](https://docs.anthropic.com/en/docs/claude-code)
- **beads** (`bd`) — [github.com/steveyegge/beads](https://github.com/steveyegge/beads)
- **tmux** — `sudo apt install tmux`

## From zip

```bash
unzip claudemuppet.zip -d ~
cd ~/claudemuppet
./bin/muppet-setup
```

## From git

```bash
git clone <this-repo> ~/claudemuppet
cd ~/claudemuppet
./bin/muppet-setup
```

## What setup does

1. Verifies prerequisites (Claude Code, beads, tmux)
2. Runs `bd setup claude` (beads hooks for Claude Code)
3. Enables Agent Teams in `~/.claude/settings.json`
4. Symlinks `tmux-claude`, `muppet-launch`, `muppet-attach` to `~/bin/`
5. Creates `config/hosts.conf` (default: local only)

To check prerequisites without installing: `./bin/muppet-setup --check-only`

**Note:** `~/bin/` must be on your PATH. Most Ubuntu/Debian systems include it by default when the directory exists. If commands aren't found after setup, add to `~/.bashrc`:
```bash
export PATH="$HOME/bin:$PATH"
```

## Add remote hosts (optional)

Edit `config/hosts.conf` to add SSH host aliases where muppets can run:
```
local
myserver
devbox
```

## First use

```bash
# Start Claude Code in tmux
tmux-claude

# Or launch a muppet
cp prompts/TEMPLATE.txt prompts/my-task.txt
# edit my-task.txt with your task details
muppet-launch local my-task prompts/my-task.txt ~/my-project

# Attach to a running muppet
muppet-attach
```
