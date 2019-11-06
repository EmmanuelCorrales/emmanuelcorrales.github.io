---
layout: post
title:  "How to remove Git submodules"
date:   2019-10-11 08:30:00 +0800
categories:  git
tags: [ git ]
---
Here's a simple guide on how to remove Git submodules. Lets say I want to remove a Git submodule called **drafts**.

1. Edit **.gitmodules** to remove lines that are similar to the example below.
```bash
[submodule "drafts"]
  path = drafts
  url = git://github.com/emcorrales/drafts.git
```

2. Stage the **.gitmodules** file.
```bash
git add .gitmodules
```
3. Remove the submodule from the git index. Ensure that there is no trailing slash. We're deleting the reference to the submodule and not the directory.
```bash
git rm --cached drafts
```
4. Remove the untracked files left by the submodule.
```bash
rm -rf drafts
```
5. Commit changes.
```bash
git commit -m "Remove draft submodule.".
```
6. There are still references to the drafts submodule. Edit **.git/config** and remove lines that are similar to the example below.
```bash
[submodule "drafts"]
  url = git://github.com/emcorrales/drafts.git
```
7. Remove the copy of the submodule's git repo.
```bash
rm -rf .git/modules/drafts
```
There you have it, you have succesfully removed the submodule from your local
git repo.
