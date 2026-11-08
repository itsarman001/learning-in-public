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
