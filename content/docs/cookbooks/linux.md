---
title: 'Linux Cookbook'
type: page
tags: ['linux']
draft: True
---

# Apt

## Installed packages

```bash
$ apt list --installed
```

## Dependencies

```bash
# Dependencies
$ apt-cache depends packagename

# Reverse dependencies 
$ apt-cache rdepends packagename
```