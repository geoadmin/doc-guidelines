# GIT Cheat Sheet

- [Status and Config](#status-and-config)
- [Update repository from remote](#update-repository-from-remote)
- [Branching](#branching)
- [Git History](#git-history)
- [Diff and merge tool](#diff-and-merge-tool)
- [Commiting](#commiting)
- [Merge, Rebase and Merge Tool](#merge-rebase-and-merge-tool)
  - [Mergetool LOCAL BASE REMOTE](#mergetool-local-base-remote)
- [Reverting changes](#reverting-changes)
- [Clean up](#clean-up)
- [Push change on remote](#push-change-on-remote)

## Status and Config

| Command | Description |
|---------|-------------|
| `git status` | Shows the status of the current repository |
| `git --no-pager config --global -l` or `git config --global -l` | List the global git configuration |
| `git --no-pager config -l` or `git config -l` | List the current repository configuration |

See also [GIT Configuration](./GIT_FLOW.md#git-configuration).

## Update repository from remote

| Command | Description |
|---------|-------------|
| `git fetch` | Fetch all remotes |
| `git fetch <remote>` | Fetch given remote |
| `git fetch -p` | Fetch all remotes (`-p` removes remotes branch that have been deleted) |
| `git pull` | Fetch remote and integrate changes into local branch. Pull does either a `merge` or `rebase` based on the configuration, at swisstopo we should use `git pull --rebase`, so make sure you have `pull.rebase=true` in your git config (see [GIT Configuration](./GIT_FLOW.md#git-configuration)) |
| `git fetch origin develop:develop` | Shortcut for `git checkout develop; git pull; git checkout <previous-branch>`. NOTE: the branch develop can be changed with another branch name |

## Branching

| Command | Description |
|---------|-------------|
| `git branch` | List all local branches |
| `git branch -a` | List all local and remote branches |
| `git checkout -b <branch>` | Create and checkout a new branch based on the current branch |
| `git branch <branch>` | Create a new branch at the current position |
| `git branch -d <branch>` | Delete branch |

## Git History

| Command | Description |
|---------|-------------|
| `gitk --all &` | Open the whole git history in a graphical view |
| `git log -n` | Display the n last commits message |
| `git log --graph --all` | Show the whole git history with commit message in the command line |
| `git log --graph --all --online` | Show the whole git history without commit message in the command line |

The following command are very useful when working on a remote server with shell only, but it highly recommended to set them as alias

| Command | Description |
|---------|-------------|
| `git log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all` | Print a nice consice git history on the commande line including relative date and commit author |
| `git log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%as%C(reset) %C(bold yellow)%d%C(reset)  %C(white)%s%C(reset) %C(dim white)- %an%C(reset)' --all` | Print a nice consice git history on the command line including author date and commit author |

## Diff and merge tool

| Command | Description |
|---------|-------------|
| `git diff` | Print a unified diff of the local changes |
| `git diff -- ':!file1.txt'` | Print a unified diff excluding the file `file1.txt` from the output |
| `git difftool` | Open the local changes with an external difftool (e.g. Beyond Compare) |
| `git mergetool` | Open conflict with external 3 way difftool for conflict resolution (e.g. Beyond Compare), see also below [Merge and Rebase](./GIT_FLOW.md#merge-and-rebase) |

## Commiting

| Command | Description |
|---------|-------------|
| `git add .` | Add all files into staging area |
| `git add <file>` | Add file(s) into staging area,  file can contain wild card |
| `git add -p <file>` | Add only parts of a file into staging area |
| `git commit --amend` | Change last commit message and/or add staging area to last commit |
| `git commit -m "<message>"` | Commit the staging area with the given message |
| `git commit` | Commit the staging area, the default editor is then popup for the commit message |

## Merge, Rebase and Merge Tool

| Command | Description |
|---------|-------------|
| `git rebase develop` | Rebase the current branch on top of develop |
| `git rebase -i develop` | Interactively rebase all commits up to the branch develop |
| `git rebase -i HEAD~3`| Interactively rebase the last 3 commits |
| `git reset --hard ORIG_HEAD` | Revert the last rebase. :warning: Create a temporary branch first for safety. |
| `git merge <branch>` | Merge the given branch into the current one |
| `git mergetool` | Open conflict with external 3 way difftool for conflict resolution (e.g. Beyond Compare) |

### Mergetool LOCAL BASE REMOTE

`git rebase`

- `LOCAL` = the base you're rebasing onto
- `REMOTE` = the commits you're moving up on top

`git merge`

- `LOCAL` = the original branch you're merging into
- `REMOTE` = the other branch whose commits you're merging in

In other words, `LOCAL` is always the original, and `REMOTE` is always the guy whose commits weren't there before, because they're being merged in or rebased on top

## Reverting changes

| Command | Description |
|---------|-------------|
| `git checkout -- .` | Revert all local changes. WARNING CHANGES ARE THEN LOST ! |
| `git reset HEAD~` | Revert last commit to the working area |
| `git reset --soft HEAD~` | Revert last commit into the staging area |
| `git reset --hard <branch|commit|remote>` | Revert all locals, staging and commits up to the given branch or commit or remote. ***WARNING CHANGES ARE THEN LOST !*** |

## Clean up

| Command | Description |
|---------|-------------|
| `git clean -xdfn` | Dry run clean the whole repository and print what would be deleted |
| `git clean -xdf` | Clean the whole repository, and bring the repo back to a fresh clone. ***WARNING THIS CANNOT BE REVERTED AND DATA MIGHT BE LOST ! CONSIDER USING `-xdfn` FIRST.*** |

## Push change on remote

| Command | Description |
|---------|-------------|
| `git push origin HEAD` | Push the current branch to the remote origin |
| `git push origin <branch>` | Push the branch to the remote origin |
| `git push origin <branch> -f` | Force push the branch to the remote origin. ***WARNING DO THIS ONLY ON PERSONAL BRANCH*** |
| `git push` | By default, it push the current branch to the remote origin. ***WARNING: Use with caution, the default behavior might change due to global configuration or branch configuration, consider using `git push origin <branch>` instead.*** |
