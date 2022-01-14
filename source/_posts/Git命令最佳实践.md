---
title: Git命令最佳实践
date: 2018-10-5 22:43:00
updated: 2018-3-5 22:43:00
comments: true
tags:
- Git
copyright: true
photos: 'https://objects.yongtao.wang/images/20181006/GitHub-cheat-sheet-graphic-v1.jpg'
---

### Create a repository

From scratch -- create a new local repository

``` bash
$ git init [project_name]
```

<!--more-->

Download form an existing repository

``` bash
$ git clone my_url
```

### Observe your repository

List new or modified files not yet committed

``` bash
$ git status
```

Show the changes to files  not yet staged

``` bash
$ git diff
```

Show the changes to staged files

``` bash
$ git diff --cached
```

Show all staged and unstaged file changes

``` bash
$ git diff HEAD
```

Show the changes between two commit ids

``` bash
$ git diff commit1 commit2
```

List the change dates and authors for a file

``` bash
$ git blame [file]
```

Show the file changes for a commit id and/or file

``` bash
$ git show [commit_id]:[file]
```

Show full change history

``` bash
$ git log
```

show change history for file/directory including diffs

``` bash
$ git log -p [file/directory]
```

### Working with branches

List all local branches

``` bash
$ git branch
```

List all branches, local and remote
``` bash
$ git branch -av
```

Switch to a branch, and update working directory

``` bash
$ git checkout my_branch
```

Create a new branch called new_branch

``` bash
$ git branch new_branch
```

Delete the branch called my_branch

``` bash
$ git branch -d my_branch
```

Merge branch_a into branch_b

``` bash
$ git checkout branch_b
$ git merge branch_a
```

Tag the current commit
``` bash
$ git tag my_tag
```

### Make a change

Stage the file, ready for commit

``` bash
$ git add [file]
```
Stage all changed files, ready for commit

``` bash
$ git add .
```

Commit all staged files to versioned history

``` bash
$ git commit -m "commit message"
```

Commit all your tracked files to versioned history

``` bash
$ git commit -am "commit message"
```

Unstages file, keeping the file changes

``` bash
$ git reset [file]
```

Revert everything to the last commit

``` bash
$ git reset --hard
```

### Synchronize

Get the latest changes from origin (no merge)

``` bash
$ git fetch
```

Fetch the latest changes from origin and merge

``` bash
$ git pull
```
Fetch the latest changes from origin and rebase

``` bash
$ git pull --rebase
```

Push local chages to the origin

``` bash
$ git push
```

### Finally

When in doubt, use git help

``` bash
$ git command --help
```

Or visit https://training.github.com/ for official GitHub training.

参考：
[Git Commands and Best Practices Cheat Sheet](https://zeroturnaround.com/rebellabs/git-commands-and-best-practices-cheat-sheet/)，Simon Maple