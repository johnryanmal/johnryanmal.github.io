---
title: Git Aliases
date: 2023-01-19 17:54:00 -0800
categories: [Snippets, Scripting]
tags: [git, shell]
---

---
Around a week ago, I found out about [git aliases](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases). And at that moment, I sold my soul to the scripting gods (willingly) to get... a customized git!

<small>A fair tradeoff, if you ask me.</small>

---
See my full gist [here](https://gist.github.com/johnryanmal/89ae7c2c26695e640c3367ef9792a0cb).

## The Basics

Aliases can be configured using `git config` like so:
```shell
$ git config --global alias.<name> <command>
```

Which allows you to run a git command:
```shell
$ git <name> <args>
git <command> <args>
```

But if you prefix `<command>` with a `!`, then it runs in the shell directly:
```shell
$ git config --global alias.<name> '!<command>'
$ git <name> <args>
<command> <args>
```

And if you also wrap it in a function (e.g. `f`), you can gain full control over the arguments:
```shell
$ git config --global alias.<name> '!f() { <command>; }; f'
$ git <name> <args>
f() { <command>; }; f <args>
```
