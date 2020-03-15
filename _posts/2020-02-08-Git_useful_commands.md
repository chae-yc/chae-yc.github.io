---
title: "Git - useful commands"
categories:
  - blogging
  - human
tags:
  - Git
last_modified_at: 2020-02-08T18:50:00+09:00
toc: true
---

1. When you forgot to add something in .gitignore

```bash
git rm -r --cached .
git add .
git commit -m "message"
git push
```
