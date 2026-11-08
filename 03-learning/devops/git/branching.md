# Git Branching

## Status

displays the current state of the working directory and staging area, helping you understand which files are staged, modified, or untracked.

```bash
git status # Default long format with detailed explanations

git status -s # Short, compact format displaying simple two-letter status codes like `M` for modified or `??` for untracked
```

- Shows staged changes that are ready to be committed and included in the next snapshot of the repository.
- Displays modified files that have not yet been staged, indicating changes made in the working directory.
- Lists untracked files that exist in the directory but are not currently tracked by Git.

```bash
➜  code-git git:(main) ✗ git status -s
 M README.md              # Modified but NOT staged. (Changes won't be committed yet)
 D git-script.sh          # Deleted but NOT staged. (Git still thinks the file exists)
AM src/product.js         # New file added to staging, then edited again afterward.
M  src/user.js            # Modified and fully staged. (Ready to be committed)
MM utils/helpers.js       # Tracked file modified, staged, then edited again afterward.
?? docs/git-cheatsheet.md # Untracked file. (Brand new file, Git is completely ignoring it)
```

## Log

`git log` Git gives you a standard and detailed history of your project. Each commit entry displays:

- The **Commit Hash** (a unique 40-character ID code)
- The **Author** (who made the change)
- The **Date and Time** of the commit
- The **Commit Message** explaining what was changed

The standard log can quickly become too long and hard to read. Adding the `--oneline` flag compresses the history into a **highly compact, one-line summary** per commit.

It simplifies the output by:

1. Shortening the long 40-character hash into a **7-character prefix**.
2. Stripping out the Author and Date.
3. Placing the hash and the **Commit Message** on a single line.

```bash
git log # Standard and detailed history
git log --oneline # Compact and one line summary of each commit
```

## Difference between `git log` and `git status`

`git status` inspects your current workspace for unsaved, real-time changes,
whereas `git log --oneline` reviews the chronological history of your permanently saved commits.

