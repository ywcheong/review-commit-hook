# Commit Review Hook

A blocking pre-commit hook that shows a dialog for every commit, allowing you to approve or reject commits before they're created. Supports multiple strategies (Zenity GUI and terminal prompt) with automatic fallback.

## Features

- **Blocking**: All commits are blocked until you approve them
- **Multiple Strategies**: Tries Zenity GUI first, falls back to terminal prompt
- **Strategy Pattern**: Easily extensible with new review strategies
- **Review on Reject**: Rejected commits allow you to leave review comments in unstaged area for AI agent

## Requirements

- Ubuntu/Debian Linux
- Zenity (GTK+ dialogs): `sudo apt install zenity` (optional, will fall back to terminal if unavailable)
- Git

## Installation

### Automated Installation

```bash
cd /path/to/your/git/repository
/path/to/review-commit-hook/install
```

This will:
1. Ask for installation directory (default: `utility/hook/`)
2. Install scripts to the specified directory
3. Create or update `.git/hooks/pre-commit` in your repository
4. Backup existing pre-commit hook if present

### Manual Installation

1. Copy scripts to your desired location:
   ```bash
   TARGET_DIR="utility/hook"
   mkdir -p "${TARGET_DIR}/strategy"
   
   cp pre-commit-hook "${TARGET_DIR}/"
   cp request-review-to-user "${TARGET_DIR}/"
   cp strategy/1-ubuntu-zenity "${TARGET_DIR}/strategy/"
   cp strategy/2-universal-terminal "${TARGET_DIR}/strategy/"
   
   chmod +x "${TARGET_DIR}/pre-commit-hook"
   chmod +x "${TARGET_DIR}/request-review-to-user"
   chmod +x "${TARGET_DIR}/strategy/1-ubuntu-zenity"
   chmod +x "${TARGET_DIR}/strategy/2-universal-terminal"

2. Create `.git/hooks/pre-commit` in your repository:
   ```bash
   #!/bin/bash
   # Review Commit Hook
   # INSTALL_PATH=/path/to/utility/hook
   
   if [ -x "${INSTALL_PATH}/pre-commit-hook" ]; then
       "${INSTALL_PATH}/pre-commit-hook"
       exit $?
   fi
   
   exit 0
   ```

3. Make it executable:
   ```bash
   chmod +x .git/hooks/pre-commit
   ```

## Usage

Once installed, every `git commit` will:

1. Try Zenity GUI (if available):
   - Show dialog with project name, branch, and staged files
   - Wait for approval or rejection

2. Fall back to terminal prompt (if GUI unavailable):
   - Display commit information in terminal
   - Ask for approval with `[y/N]` prompt

3. Wait for your decision:
   - **Approve Commit**: Commit proceeds
   - **Request Changes**: Commit is blocked; leave review comments in unstaged area

## Bypassing the Hook

To skip the review temporarily:

```bash
git commit --no-verify -m "message"
```

## Uninstallation

Run the uninstall script from within a git repository:

```bash
/path/to/review-commit-hook/uninstall
```

This will:
1. Read the installation path from `.git/hooks/pre-commit`
2. Remove all installed files
3. Clean up empty directories
4. Restore original pre-commit hook from backup (if exists)

## How It Works

```
git commit
    │
    ▼
.git/hooks/pre-commit
    │
    ▼
pre-commit-hook
    │
    ▼
request-review-to-user
    │
    ├── Check for staged changes
    │   └── No staged changes → exit 0 (skip)
    │
    ├── Try strategy/1-ubuntu-zenity
    │   ├── Zenity available → Show GUI
    │   │   ├── Approve → exit 0
    │   │   └── Reject → exit 1
    │   └── Zenity unavailable → exit 2 (try next)
    │
    ├── Try strategy/2-universal-terminal
    │   ├── GUI Terminal available → Open new window
    │   │   ├── y/Y → exit 0
    │   │   └── Other → exit 1
    │   └── No GUI Terminal → exit 2
```

## File Structure

```
review-commit-hook/
├── install                      # Installation script
├── uninstall                    # Uninstallation script
├── pre-commit-hook              # Entry point for .git/hooks/pre-commit
├── request-review-to-user       # Strategy orchestrator
└── strategy/
    ├── 1-ubuntu-zenity          # Zenity GUI strategy
    └── 2-universal-terminal  # Universal GUI Terminal strategy

## Exit Code Convention

| Code | Meaning | Behavior |
|------|---------|----------|
| 0 | USER APPROVED | Commit proceeds |
| 1 | USER REJECTED | Commit blocked, show message to AI |
| 2 | UNAVAILABLE | Try next strategy |

## Extending with New Strategies

To add a new review strategy:

1. Create a new file in `strategy/` directory (e.g., `3-macos-osascript`)
2. Make it executable
3. Follow the exit code convention:
   - Return `0` if user approves
   - Return `1` if user rejects
   - Return `2` if strategy is unavailable
4. Add the strategy name to the `STRATEGIES` array in `request-review-to-user`

## Troubleshooting

### "zenity not found"

Install zenity:
```bash
sudo apt install zenity
```

Or use the terminal fallback (automatic).

### How to leave review comments for AI agent

When you reject a commit:
1. Make your changes to the files
2. Do NOT stage them (don't run `git add`)
3. The AI agent will read your unstaged changes as review feedback

### Hook not running

1. Check `.git/hooks/pre-commit` exists and is executable:
   ```bash
   ls -la .git/hooks/pre-commit
   ```
2. Verify the `INSTALL_PATH` in the hook points to the correct location:
   ```bash
   grep "INSTALL_PATH" .git/hooks/pre-commit
   ```
3. Check the hook scripts are executable:
   ```bash
   ls -la utility/hook/
   ```

### All strategies unavailable

If all strategies return exit code 2, the hook will allow the commit to proceed (exit 0). This prevents the hook from blocking commits in non-interactive environments (CI/CD, scripts, etc.).
