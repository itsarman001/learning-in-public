# Git Branching

Git branching allows you to maintain separate lines of development within a repository. It is commonly used to develop features, fix bugs, and experiment without directly affecting the main codebase.

## Status

`git status` displays the current state of the working directory and staging area, helping you identify which files are modified, staged, deleted, or untracked.

### Commands

```
git status       # Detailed status with explanations
git status -s    # Short, compact status
git status -sb   # Short status with branch information
```

### Understanding Status Codes

Git uses two columns to represent the state of a file:

|Code|Meaning|
|---|---|
|`M`|Modified and staged|
|`M`|Modified but not staged|
|`A`|New file added to staging|
|`D`|Deleted and staged|
|`D`|Deleted but not staged|
|`R`|Renamed and staged|
|`C`|Copied and staged|
|`??`|Untracked file|
|`AM`|Added to staging, then modified|
|`MM`|Modified, staged, then modified again|

### Example

```
git status -s

 M README.md              # Modified but not staged
 D git-script.sh          # Deleted but not staged
AM src/product.js         # Added, then modified
M  src/user.js            # Modified and staged
MM utils/helpers.js       # Modified, staged, then modified again
?? docs/git-cheatsheet.md # Untracked file
```

## Log

`git log` displays the commit history of a repository. Each commit contains information such as its hash, author, date, and commit message.

### Commands

```
git log                         # Detailed commit history
git log --oneline                # Compact commit history
git log --oneline --graph --all  # Visual history of all branches
git log -n 5                     # Show the latest 5 commits
git log --stat                   # Show files changed in each commit
```

The `--oneline` flag displays each commit in a single line, using an abbreviated commit hash followed by its commit message.

**Difference between** `**git status**` **and** `**git log**`**:**

- `git status` shows the current state of your working directory and staging area.
    
- `git log` shows the history of previously committed changes.
    

## Creating and Switching Branches

A branch is a movable pointer to a commit. It allows you to work on changes independently without affecting other branches.

### Commands

```
git branch                    # List local branches
git branch <branch_name>      # Create a new branch
git switch <branch_name>      # Switch to an existing branch
git switch -c <branch_name>   # Create and switch to a new branch
git branch -m <new_name>      # Rename the current branch
```

## Merging Techniques

Merging integrates changes from one branch into another. For example, when a feature is complete, its changes can be merged into `main`.

```
git switch main
git merge feature
```

Git provides different approaches to integrate branch changes.

### 1. Fast-Forward Merge

A fast-forward merge occurs when the target branch has no new commits since the feature branch was created.

Git simply moves the target branch pointer forward to the latest commit of the feature branch. No additional merge commit is created.

```
git switch main
git merge feature
```

**Characteristics:**

- Does not create a merge commit.
    
- Maintains a linear commit history.
    
- Only possible when the target branch is an ancestor of the incoming branch.
    

### 2. Three-Way Merge

A three-way merge occurs when both branches have progressed independently.

Git uses three points to combine the changes:

1. Common ancestor: The commit where the branches diverged.
    
2. Current branch: The branch receiving the changes.
    
3. Incoming branch: The branch being merged.
    

Git compares these commits and creates a new merge commit containing the combined changes.

```
git switch main
git merge feature
```

**Characteristics:**

- Creates a merge commit.
    
- Preserves the history of both branches.
    
- Useful when branches contain independent commits.
    
- May require manual conflict resolution.
    

To explicitly create a merge commit even when a fast-forward is possible:

```
git merge --no-ff feature
```

### 3. Squash Merge

A squash merge combines all changes from the incoming branch into a single staged change instead of preserving its individual commits in the target branch history.

```
git switch main
git merge --squash feature
git commit -m "feat: add authentication"
```

**Characteristics:**

- Combines multiple changes into one commit.
    
- Does not create a merge commit.
    
- Keeps the target branch history cleaner.
    
- Individual feature branch commits are not preserved in the target branch history.
    

**Use case:** When a feature branch contains several small or intermediate commits that should be represented by one meaningful commit.

### 4. Rebase and Merge

Rebase moves the feature branch commits onto the latest commit of the target branch by replaying them one by one.

```
git switch feature
git rebase main

git switch main
git merge feature
```

After rebasing, the feature branch is typically ahead of `main`, allowing Git to perform a fast-forward merge.

**Characteristics:**

- Produces a linear commit history.
    
- Avoids unnecessary merge commits.
    
- Rewrites commit history and creates new commit hashes.
    
- Makes feature development appear as if it happened on top of the latest target branch.
    

**Note:** Avoid rebasing commits that have already been shared with others unless the team has agreed to rewrite that history.

### 5. Recursive Merge Strategy

The recursive strategy was Git's traditional default strategy for merging two branches using a three-way merge.

```
git merge -s recursive feature
```

It could recursively combine multiple common ancestors when branches had more complex histories.

**Note:** Since Git 2.33, `ort` is the default strategy for ordinary two-branch merges. The recursive strategy is now largely historical.

```
git merge -s ort feature
```

### Quick Comparison

|Technique|Merge commit|History|
|---|---|---|
|Fast-forward|No|Linear|
|Three-way|Yes|Preserves branching|
|Squash|No|Linear|
|Rebase and merge|Usually no|Linear|
|Recursive strategy|Depends|Depends on merge|

**Note:** Fast-forward, three-way, squash, and rebase describe different integration approaches. Recursive and `ort` are underlying merge strategies.

## Deleting a Branch

Once a feature branch has been merged and is no longer needed, it can be deleted to keep the repository organized.

### Delete a Local Branch

```
git branch -d <branch_name>
```

The `-d` flag safely deletes a branch only when Git considers it fully merged into its upstream branch or the current branch.

### Force Delete a Local Branch

```
git branch -D <branch_name>
```

The `-D` flag forcefully deletes a branch even if it contains unmerged commits.

**Warning:** Unmerged changes may become difficult to recover after deleting the branch.

### Delete a Remote Branch

```
git push origin --delete <branch_name>
```

This removes the branch from the remote repository, such as GitHub.

**Important:**

- You cannot delete the branch you are currently on.
    
- Deleting a local branch does not delete its remote counterpart.
    
- Deleting a remote branch does not automatically delete your local branch.
    

## Stash

Git Stash temporarily saves uncommitted changes without creating a commit. It is useful when you need to switch branches while keeping your current work for later.

For example, suppose you are working on branch `A` and need to switch to branch `B` to fix an urgent issue. If your uncommitted changes conflict with the target branch, Git may prevent you from switching.

Instead of committing incomplete work or discarding your changes, you can stash them and restore them later.

### Commands

```
git stash                          # Save tracked changes
git stash push -m "WIP: feature"   # Save with a descriptive message
git stash -u                       # Include untracked files
git stash list                     # List saved stashes
git stash show                     # Show latest stash summary
git stash show -p                  # Show actual changes
git stash apply                    # Restore latest stash
git stash pop                      # Restore and remove latest stash
git stash apply stash@{1}          # Apply a specific stash
git stash drop stash@{0}           # Remove a specific stash
git stash clear                    # Remove all stashes
```

### Example Workflow

```
# Working on branch A
git stash push -m "WIP: authentication"

# Switch to another branch
git switch B

# Complete the required work
git add .
git commit -m "feat: update dashboard"

# Return to the original branch
git switch A

# Restore previous changes
git stash pop
```

**Important:**

- `git stash apply` restores changes but keeps the stash.
    
- `git stash pop` restores changes and removes the stash if successful.
    
- Untracked files are excluded by default. Use `-u` to include them.
    
- Use `-a` to include ignored files as well.
    
- Stashes are stored locally and are not pushed to remote repositories.
    
- Restoring a stash can result in conflicts if the branch has changed.