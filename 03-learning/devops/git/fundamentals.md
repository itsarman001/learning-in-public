# Git Fundamentals

## Git Config

```bash
# Set global name and email
git config --global user.name "John Doe"
git config --global user.email "johndoe@email.com"

# Set local (per-repo) name and email
git config user.name "John Doe"
git config user.email "johndoe@email.com"

# Set default editor
git config --global core.editor "code --wait"

# Line endings: true (Windows) | input (Linux/macOS)
git config --global core.autocrlf input

# Edit configs
git config --global -e

# Start SSH agent on (Linux/macOS)
eval "$(ssh-agent -s)"

# Add an SSH key on (Linux/macOS)
ssh-add ~/.ssh/id_ed25519
```

```powershell
# Start SSH agent (run PowerShell as Administrator)
# One-time: make the service start automatically
Set-Service -Name ssh-agent -StartupType Automatic

# Start the service
Start-Service ssh-agent

# Check that it's running
Get-Service ssh-agent

# Add an SSH key
ssh-add $env:USERPROFILE\.ssh\id_ed25519

# List loaded keys
ssh-add -l
```

## Initialize Git in a Project

```bash
git init
```

## File Status Tags

| `git status -s` | VS Code | Meaning |
| --------------- | ------- | ------- |
| `??`            | U       | Untracked |
| `A`             | A       | Added (staged) |
| `M`             | M       | Modified |
| `D`             | D       | Deleted |
| `R`             | R       | Renamed |

```bash
git status -s      # short status of staged/unstaged files
git log --oneline  # compact commit history
```

## Undo Commits with `git reset`

```bash
# git reset --<option> HEAD~n
# n = number of commits to go back (1, 2, 3...)
git reset --soft HEAD~1
```

| Option              | Commit undone | Changes stay staged | Working files kept |
| ------------------- | ------------- | ------------------- | ------------------ |
| `--soft`            | Yes           | Yes                 | Yes                |
| `--mixed` (default) | Yes           | No (unstaged)       | Yes                |
| `--hard`            | Yes           | No                  | **No, discarded**  |

> ⚠️ `--hard` permanently deletes uncommitted changes. Use with care.