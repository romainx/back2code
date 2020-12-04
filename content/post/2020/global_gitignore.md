---
title: 'Global gitignore'
date: '2020-12-03'
categories: [dev]
tags: ['git']
---

Here is a short but very useful tip to avoid committing **unwanted files** by configuring a `gitignore` at **global level**. The main advantage is that it's configured once for all and you don't have to do it on each project.

<!--more-->

## Basic example

This one-liner is a good starting point for Mac users.

```bash
printf ".DS_Store\\n.vscode\\n" > ~/.gitignore && \\
	git config --global core.excludesfile ~/.gitignore
```

## References / Further reading

- [sebastiandedeyne.com](https://sebastiandedeyne.com/setting-up-a-global-gitignore-file/)