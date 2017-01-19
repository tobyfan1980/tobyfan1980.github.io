---
layout: post
title:  "Working on Github branches"
date:   2017-01-19 10:00:00 +0200
categories:    Tools
tags:    Github
---

1 -- Working on new branch

1) create branch locally

```
git branch <branch name>
git checkout <branch name>
```

2) make changes

3) commit changes and create remote branch

```
git add .
git commit -m "change message"
git push origin <branch name>
```

4) switch to master

```
git checkout master
git pull origin master
```

5) merge branch to master

```
git merge <branch name>
```

6) resolve conflict if any
make correction

```
git add .
git commit -m "resolve message"
git push origin master
```

7) delete local branch

```
git branch -d <branch name>
```

8) delete remote branch

```
git push origin --delete <branch name>
```

2 -- modify committed changes (not pushed)

2.1 modify commit message only

```
git commit --amend -m "new message"
```

2.2 modify commit 

add new file

```
git commit --amend
```
(new file will be merged to current commit)

2.3 move commit from branch1 to branch2

1) copy commit

```
git checkout branch2
git cherry-pick <commit hash>
```

2) remove commit from branch1

```
git checkout branch1
git reset --hard <previous commit hash>
```
(can choose --soft which only remove the commit metadata, or no argument which pull all code from staging area)

```
git clean
```
(if there are untracked code)

2.4 uncommit a change (pushed or not)

1) make sure there is no local change

2) revert 

```
git revert <commit hash>
git push origin <branch name>
```
