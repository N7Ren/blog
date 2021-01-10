---
title: How to show git branches in terminal
date: "2019-12-14T08:00:00.000Z"
template: post
draft: false
slug: "show-git-branches-in-terminal"
category: "git"
tags:
  - "git"
  - "bash"
description: "Display current git branch in bash"
socialImage: "/media/terminal-git-show-branches.png"
---

One of the first things I do when setting up a new dev system is activate displaying git branches in the terminal. I find it very practical to know on which branch I'm working on. To show the branch in the console you need to replace the following code in `~/.bashrc`:
```bash
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt
```
with this:

```bash
parse_git_branch() {
 git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
}
if [ "$color_prompt" = yes ]; then
 PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[01;31m\]$(parse_git_branch)\[\033[00m\]\$ '
else
 PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w$(parse_git_branch)\$ '
fi
```
Finally, to activate the functionality you have to call `source ~/.bashrc` in the terminal:
![Terminal showing git branch](/media/terminal-git-show-branches.png)

Source: <a href="https://askubuntu.com/questions/730754/how-do-i-show-the-git-branch-with-colours-in-bash-prompt" target="_blank">https://askubuntu.com/questions/730754/how-do-i-show-the-git-branch-with-colours-in-bash-prompt</a>
