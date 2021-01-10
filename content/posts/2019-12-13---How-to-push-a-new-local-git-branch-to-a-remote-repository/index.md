---
title: How to push a new local git branch to a remote repository
date: "2019-12-13T09:00:00.000Z"
template: post
draft: false
slug: "push-new-local-git-branch-to-remote"
category: "git"
tags:
  - "git"
description: "Push new git branch to remote"
socialImage: "/media/git-push-new-local-branch.png"
---

First create a new git branch with `git checkout -b [new-branch-name]`. Then push it with `git push -u origin [new-branch-name]`. The `-b` option in `git checkout` is a shortcut for two commands. It's the same as:
```bash
git branch [new-branch-name] 
git checkout [new-branch-name]
```

The `-u` option in `git push` is a shorter version of `--set-upstream` which allows git to track the *upstream branch*. This removes the need to specify the branch name in further git commands. 

Example:
![Push new git branch to remote repository](/media/git-push-new-local-branch.png)


Source: <a href="https://www.freecodecamp.org/forum/t/push-a-new-local-branch-to-a-remote-git-repository-and-track-it-too/13222" target="_blank">https://www.freecodecamp.org/forum/t/push-a-new-local-branch-to-a-remote-git-repository-and-track-it-too/13222</a>