---
layout: post
title:  "How to remove Git submodules."
date:   2019-10-11 08:30:00 +0800
categories:  git
tags: [ git ]
---
I decided to write this post because I've been getting a lot of tickets
regarding pipelines not working due to improper removal of git submodule/s in
their repo. Here's a simple guide on how to remove Git submodules.

Lets say I want to remove a Git submodule called **drafts**.

1. Edit **.gitmodules** to remove lines that are similar to the example below.
```bash
[submodule "drafts"]
  path = drafts
  url = git://github.com/emcorrales/drafts.git
```

2. Edit **.git/config** and remove lines that are similar to the example below.
```bash
[submodule "drafts"]
  url = git://github.com/emcorrales/drafts.git
```

3. Remove the submodule from the git index.
```bash
git rm --cached drafts
```

4. Remove submodule's git repo. The command above will move the **.git**
directory of the submodule **drafts** to **.git/modules/drafts** to protect its
history so we remove it as well.
```bash
rm -rf .git/modules/drafts
```

5. Remove submodule directory because git sees ./drafts as a new directory
instead of a submodule.
```bash
rm -rf drafts
```

6. Commit changes then push.
```bash
git add .
git push origin master
```

There you have it, You have succesfully removed the submodule from your local
git repo.
