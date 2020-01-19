---
title: 'Sphinx'
date: '2020-01-17'
draft: True
categories: [dev]
tags: ['python']
---

```bash
pip install poetry
```

```bash
$ poetry new spinx-demo
$ cd spinx-demo
$ poetry add -D sphinx ghp-import
```

```bash
sphinx-quickstart

docs
githubpages: create .nojekyll file to publish the document on GitHub pages (y/n) [n]: y
cd docs
make html
python -m http.server
make clean