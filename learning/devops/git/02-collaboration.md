# Collaboration

Git collaboration allows multiple developers to work on the same repository, share changes, and integrate their work without directly overwriting each other's contributions.

## Remote Repositories

A remote is a reference to a repository hosted elsewhere, such as GitHub. It allows developers to push and fetch changes between local and remote repositories.

```
git remote -v                    # List configured remotes
git remote add origin <url>      # Add a remote repository
git remote remove origin         # Remove a remote
git remote show origin           # Show remote details
```

## Push

`git push` uploads local commits to a remote repository.

```
git push origin main
git push -u origin feature/auth  # Push and set upstream
```

## Fetch

`git fetch` downloads new commits and references from a remote repository without integrating them into your current branch.

```
git fetch origin
git fetch --all
git fetch --prune                # Remove stale remote-tracking references
```

## Pull

`git pull` fetches remote changes and integrates them into the current branch.

```
git pull origin main
git pull --rebase origin main
```

By default, Git uses the configured pull strategy, which may be merge, rebase, or fast-forward only.

## Clone

`git clone` creates a local copy of an existing remote repository.

```
git clone <repository_url>
git clone <repository_url> <directory_name>
```

## Fork

A fork is an independent copy of another user's repository under your own GitHub account.

It is commonly used when contributing to projects where you don't have direct write access.

## Pull Requests (PR)

A pull request is a GitHub feature used to propose changes from one branch or repository into another.

Typical workflow:

1. Create a feature branch or fork the repository.
    
2. Make changes and commit them.
    
3. Push your branch to the remote.
    
4. Open a pull request targeting the appropriate branch.
    
5. Review changes and address feedback.
    
6. Merge the pull request after approval.
    

## Upstream Repository

In fork-based workflows, `origin` usually refers to your fork, while `upstream` refers to the original repository.

```
git remote add upstream <original_repository_url>
git remote -v

git fetch upstream
```

To update your local main branch with changes from the original repository:

```
git switch main
git merge upstream/main
```

Alternatively, use `git rebase upstream/main` when a linear history is preferred.

## Merge Conflicts

Merge conflicts occur when Git cannot automatically combine changes from different branches. Resolve the conflicting sections, stage the affected files, and complete the merge or rebase.

```
git status
git add <file_name>
git merge --continue       # During a merge
git rebase --continue      # During a rebase
```