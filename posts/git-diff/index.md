---
title: Git diff
slug: git-diff
description: Understand the git diff command and how to compare changes in Git
language: en
is_favorite: false
tags:
  - git
---

these days which codes are written by Agents 
i think its good to have review skills and just read the codes which is generated
so  i think knowing git diff well is very handy

## what does `diff A B` mean ?

Think of `A` and `B` as two files. `diff A B` shows the changes are needed to turn `A` into `B` (`A -> B`).
-  `-` lines exist in `A` but need to be removed to transform `A` into `B`. They are in `A` and not in `B`.
- `+` lines don't exist in `A` and need to be added to transform `A` into `B`. They are in `B` and not in `A`.

![Diff between file A and file B showing removed and added lines](assets/img1.png)

That is the whole concept. `A` is where you start, `B` is where you end up, and the diff describes how to get from `A` to `B`.
You can test this concept right now without a Git repository. Git can diff any two plain files on your system:

```bash
git diff --no-index a.txt b.txt
```

No repo, no commits, no staging area. Just `A` and `B`.

## Reading a diff
...

## The defaults

In everyday use `A` and `B` are rarely physical files on disk. They represent states in your repository such as commits, branches, the staging area, or your working directory.
The `A -> B` concept remains but you rarely need to specify both sides explicitly because Git infers them using smart defaults.

### `git diff`

- **A (start):** the staging area
- **B (end):** the working directory
- **Transform:** staging area -> working directory
- **Shows:** changes not yet staged
- **Question it answers:** what have i changed since my last `git add`?

### `git diff --staged` (or `git diff --cached`)

- **A (start):** HEAD
- **B (end):** the staging area
- **Transform:** HEAD -> staging area
- **Shows:** the staged changes ready for commit
- **Question it answers:** what changes am i about to commit?

### `git diff HEAD`

- **A (start):** HEAD
- **B (end):** the working directory
- **Transform:** HEAD -> working directory
- **Shows:** all changes (both staged and unstaged) since the last commit
- **Question it answers:** how is my working tree different from my last commit?


![Git diff comparing HEAD, the staging area and the working directory](assets/img2.png)


## `git diff A B` vs `A..B` vs `A...B`

`git diff A..B` is equivalent to `git diff A B` both show the changes needed to transform `A` into `B`.

`git diff A...B` is equivalent to `git diff $(git merge-base A B) B`.
It compares the **common ancestor** of `A` and `B` with `B`, showing only the changes made on `B` since it branched off from `A`.

**Comparing `main` and `feature`:**

```
       c3---c4---c5  (main)
      /
c1---c2
      \
       f1---f2---f3  (feature)
```

- `git diff main...feature` → compares `c2` (the merge-base) with `f3` (tip of feature)
    - Shows **only** the work done on the feature branch: `f3`, `f2`, `f1`
    - This is usually what you want when reviewing a branch, because it ignores whatever landed on `main` in the meantime
- `git diff main..feature` → compares `c5` (tip of main) with `f3` (tip of feature)
    - Shows all differences between both branches


## `git show`

Shows information about a Git object (typically a commit), including:
- the commit metadata (hash, author, date, message)
- the changes (diff) that commit introduced

### The diff Behind `git show`

The diff part of `git show` compares a commit with its parent:
- **A (start):** the parent commit
- **B (end):** the commit being shown
- so `git show HEAD` shows the diff `HEAD^ -> HEAD`, and `git show <commit-hash>` shows `<commit-hash>^ -> <commit-hash>`

```sh
git show
# ↓ defaults to HEAD
git show HEAD
# ↓ under the hood, compares parent to HEAD
git diff HEAD^ HEAD

git show abc1234
# ↓ equivalent to:
git diff abc1234^ abc1234
```

## Useful Flags

```sh
# ignore whitespace changes when comparing lines
git diff -w

# compact summary of changes (files changed, insertions, deletions)
git diff --stat

# show only the names of changed files
git diff --name-only

# word-by-word instead of line-by-line
git diff --word-diff

# swap A and B
git diff -R

# only this file
git diff -- main.go
```
