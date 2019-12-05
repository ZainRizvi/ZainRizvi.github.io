---
layout: post
title: The Essential Git Cheat Sheet
excerpt: Quick cheat sheet of commonly used git commands that I’d otherwise have to
  keep Googling to remember
tags:
- cheatsheet
- github
- git

---
Quick cheat sheet of commonly used git commands that I’d otherwise have to keep Googling to remember

![](/media/2019-11-27-github.jpeg)

# Committing Changes

## Merge changes with last commit

    git commit --amend --no-edit

## Squash commits

To squash the last four commits into one, do the following:

    git rebase -i HEAD~4

or

    git rebase -i [hash of commit before the one last one you want to squash]

In the list that shows up, replace the word “pick” with “squash” or “s” next to all but the oldest commit you’re squashing

## Set the date of the last commit to right now

You may want to do this after you've squashed some commits

    git commit --amend --no-edit --date "$(date)"

# Undoing/Reverting Changes

## Discard all local changes to tracked files

    git reset --hard

## Discard all local changes to a single unstaged file

    git checkout --[file]

## Discard changes to all untracked/unstaged files. _This cannot be undone_

    # first run 'git clean -n' to get a preview of what will be deleted
    git clean -f

## Discard all local changes

Combine the above to get:

    git reset --hardgit clean -f 

## Revert/Undo an Entire Commit

Note: this leaves the commit in the branch history

    git revert [commit-id]

# Set your branch head to the remote branch’s head

Use when you want to discard any local changes and start over from the remote branch state

    # Replace 'origin/master' with your desired remote branch or 
    #   the specific CL you want to set the HEAD to
    git reset --hard origin/master 

# Pull a new remote branch to your existing repo

For when you want to pull a new branch from your repo origin.  You can replace 'origin' with your desired remote branch if you're using something different

    git checkout --track origin/[branch_name]

For more details see [this article](https://stackabuse.com/git-fetch-a-remote-branch/)

# Changing the Branch HEAD

To change the current branch’s head to a different commit

This can be used as a form of undo to remove commits from the dependency tree

    git reset --hard [commit-id]
    git push -f # This line changes the head on the remote branch as well

# Create a New Branch from Current Branch

    git checkout -b [newBranch]

# Diffing Changes

Key thing to note is that appending \~ to the end of a commit makes it refer to it’s parent. Appending \~N instead makes it refer to it’s Nth parent

## Diffing the last commit

    git diff HEAD~ HEAD

## Diffing a specific commit

    git diff [commitId]~ [commitId] #e.g. git diff f23a4s~ f23a4s

## Diffing the last N commits

    git diff HEAD~N HEAD # e.g diff last 4 commits: git diff HEAD~4 HEAD

# Pretty Git Log printout

Add the following to your [git config](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Where-system-global-and-local-Windows-Git-config-files-are-saved) file to enable pretty logs (based on work by [Filipe Kiss](https://coderwall.com/p/euwpig/a-better-git-log))

    alias.lg=!git lg1
    alias.lg1=!git lg1-specific
    alias.lg2=!git lg2-specific
    alias.lg3=!git lg3-specific
    alias.lgs=!git lg1 — simplify-by-decoration
    alias.lg1s=!git lg1-specific — all — simplify-by-decoration
    alias.lg2s=!git lg2-specific — all — simplify-by-decoration
    alias.lg3s=!git lg3-specific — all — simplify-by-decoration
    alias.lg1-specific=log — graph — abbrev-commit — decorate — format=format:’%C(bold blue)%h%C(reset) — %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(auto)%d%C(reset)’
    alias.lg2-specific=log — graph — abbrev-commit — decorate — format=format:’%C(bold blue)%h%C(reset) — %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(auto)%d%C(reset)%n’’ %C(white)%s%C(reset) %C(dim white)- %an%C(reset)’
    alias.lg3-specific=log — graph — abbrev-commit — decorate — format=format:’%C(bold blue)%h%C(reset) — %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset) %C(bold cyan)(committed: %cD)%C(reset) %C(auto)%d%C(reset)%n’’ %C(white)%s%C(reset)%n’’ %C(dim white)- %an <%ae> %C(reset) %C(dim white)(committer: %cn <%ce>)%C(reset)’

With the above in your git config file you can just enter the below to see your history:

    git lg

Or to limit history to just the past N commits:

    git lg -[# of commits]  # Example: git lg -5