---
title: "Git Cheat Sheet"
slug: git-cheat-sheet
date: 2021-11-25T11:58:06+01:00
categories:
 - Software development
tags:
 - git
images:
  - /wp-content/uploads/2014/03/Git-Logo.png
---

A simple git cheat sheet.

<!--more-->

## Commit

Change last commit message.\
`git commit --amend`

## Remote

List remotes.\
`git remote -v`

List remote branches.\
`git branch -r`

Add multiple push urls.
```bash
git remote set-url origin --push --add <a remote>
git remote set-url origin --push --add <another remote>
```

Add upstream.\
`git remote add upstream ssh@example.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git`

Update fork.\
```bash
git fetch upstream
git merge upstream/main
```

Resolve history mismatch.\
````
git fetch --all
git reset --hard origin/master
````

## Tags

List tags.\
`git tag`

Add tag.\
`git tag $TAG_NAME`

Delete tag.\
`git tag -d $TAG_NAME`

Push tags.\
`git push --tags`

Checkout tag.\
`git checkout tags/$TAG_NAME`

## Branching

Move commit to another branch.\
```bash
git checkout -b $BRANCH

git switch master
git reset --hard HEAD~1
git push -f
```

Fetch branch without checking out.\
`git fetch origin master`

Prune local remote branches .\
`git remote prune origin`

Pull all remote branches.\
```bash
git branch -r | grep -v '\->' | while read remote; do git branch --track "${remote#origin/}" "$remote"; done
git fetch --all
git pull --all
```

Delete local branches except master.\
`git branch | grep -v "master" | xargs git branch -D

Delete local branches that do not exist on remote.\
```bash
git fetch -p && git branch --merged | grep -v '\*' | xargs -n 1 git branch -d
```

## Staging

Save unstaged changes.\
```
git stash
git stash apply
```

Stash a specific file.\
`git stash file path/to/file`

Remove untracked local files and directories.\
`git clean -df`

Stash with name.\
`git stash save "$DESCRIPTION"`

## Log

Search the git log.\
`git log --all --grep='Build 0051'`

## Submodule

Create local config.\
`git submodule init`

Clone submodules.\
`git submodule update`

Initialize and pull git submodules recursively.\
`git submodule update --init --recursive`

Show remote of submodules.\
`git config --file=.gitmodules -e`

Remove a submodule.\
```bash
# Remove submodule from .git/config
git submodule deinit -f path_to_submodule

# Remove submodule from .gitmodule and remove directory 
git rm --cached path_to_submodule

# Remove submodule form .git/modules directory
rm -rf .git/modules/path_to_submodule

# Remove submodule
rm -rf path_to_submodule
```

Get current branch of submodules.\
`git submodule foreach 'git status' | grep 'On branch' -b1`

Remove all submdoules.\
```bash
# Deinit all submodules from .gitmodules
git submodule deinit .

# Remove all submodules (`git rm`) from .gitmodules
git submodule | cut -c43- | while read -r line; do (git rm "$line"); done

# Delete all submodule sections from .git/config (`git config --local --remove-section`) by fetching those from .git/config
git config --local -l | grep submodule | sed -e 's/^\(submodule\.[^.]*\)\(.*\)/\1/g' | while read -r line; do (git config --local --remove-section "$line"); done

# Manually remove leftovers
rm .gitmodules
rm -rf .git/modules
```

## Diff

Setup merge tool.\
`git config merge.tool vimdiff`

Show changed files only.\
`git diff --name-only HEAD HEAD~1`

Show changes for one file only.\
`git diff HEAD HEAD~1 path/to/file`

Show diff for commit and its ancestor .\
`git diff 0690a1046771702ec50c3ccf9e834d33debeb2f0^ 0690a1046771702ec50c3ccf9e834d33debeb2f0`

## Config

Set default editor.\
`git config --global core.editor "vim"`

## Pull Requests

You can checkout a single pull request reference by doing.\
```bash
# Replace $PR with the pull request number.
git fetch origin pull/$PR/head
git checkout FETCH_HEAD
```

If you want to fetch them all.\
```bash
git fetch origin +refs/pull/*/merge:refs/remotes/origin/pr/*
```

For which you can then checkout the pull request by doing.\
```bash
git checkout origin/pr/$PR
```

If you want to automatically fetch pull requests you can add it to your repo's config.\
```bash
git config --add remote.origin.fetch +refs/pull/*/merge:refs/remotes/origin/pr/*/merge
git fetch
```

## Rebase

Squal the last three commits.\
```
git rebase -i HEAD~3
```
Then use keywords `pick` to select final commit and `squash` for the once to be merged.

Remove zip files from git history.\
```
git filter-branch --tree-filter 'rm -f path/to/folder/*.zip' HEAD
git push origin --force --all
```