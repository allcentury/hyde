---
layout: post
title: git add --patch
---

Git is so powerful and with such a rich feature set it can take a tremendous amount of time to learn.  Over time, I've been trying to add to my git alias' with useful commands that have improved my work flow.  This is my first post on a series of commands I'd like to share.

First up is `git add`.  When I pair with a lot of other developers, I see the same work flow - `git add .`.  Readers likely know that this stages everything into the current tree/index.  To see this in action, I've run a `rails new advanced-git` command to give us some files to play with.

That generated this:

```bash
~/projects/advanced-git
▶ ls -la
total 56
drwxr-xr-x   18 anthonyross  staff   612 Jul 21 11:21 .
drwxr-xr-x  147 anthonyross  staff  4998 Jul 21 11:22 ..
-rw-r--r--    1 anthonyross  staff   543 Jul 21 11:21 .gitignore
-rw-r--r--    1 anthonyross  staff  1715 Jul 21 11:21 Gemfile
-rw-r--r--    1 anthonyross  staff  4267 Jul 21 11:21 Gemfile.lock
-rw-r--r--    1 anthonyross  staff   374 Jul 21 11:21 README.md
-rw-r--r--    1 anthonyross  staff   227 Jul 21 11:21 Rakefile
drwxr-xr-x   10 anthonyross  staff   340 Jul 21 11:21 app
drwxr-xr-x    8 anthonyross  staff   272 Jul 21 11:21 bin
drwxr-xr-x   14 anthonyross  staff   476 Jul 21 11:21 config
-rw-r--r--    1 anthonyross  staff   130 Jul 21 11:21 config.ru
drwxr-xr-x    3 anthonyross  staff   102 Jul 21 11:21 db
drwxr-xr-x    4 anthonyross  staff   136 Jul 21 11:21 lib
drwxr-xr-x    3 anthonyross  staff   102 Jul 21 11:21 log
drwxr-xr-x    9 anthonyross  staff   306 Jul 21 11:21 public
drwxr-xr-x    9 anthonyross  staff   306 Jul 21 11:21 test
drwxr-xr-x    4 anthonyross  staff   136 Jul 21 11:21 tmp
drwxr-xr-x    3 anthonyross  staff   102 Jul 21 11:21 vendor
```

Let's pretend this is an existing project that has a few commits:

```bash
git init
git add .
git commit -m "Initial Commit"
```

Now let's edit the README.md file and explore a useful git feature.  Edit the file to look like this:

```
# README

## Project Overview

This is a new project to showcase some useful git features



This is a footer where I might put license information.
```

Normally if you ran `git add .` you'd know that this file was staged with all the changes.  However, with `git add -p` you can see your changes in 'hunks'.

```bash
▶ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")

~/projects/advanced-git  master ✗
▶ git add -p
diff --git a/README.md b/README.md
index 7db80e4..fcae1bc 100644
--- a/README.md
+++ b/README.md
@@ -1,24 +1,8 @@
 # README

-This README would normally document whatever steps are necessary to get the
-application up and running.
+## Project Overview

-Things you may want to cover:
+This is a new project to showcase some useful git features

-* Ruby version

-* System dependencies
-
-* Configuration
-
-* Database creation
-
-* Database initialization
-
-* How to run the test suite
-
-* Services (job queues, cache servers, search engines, etc.)
-
-* Deployment instructions
-
-* ...
+This is a footer where I might put license information.
Stage this hunk [y,n,q,a,d,/,s,e,?]?
```

The `-p` stands for `patch` and if you run `git add --help` you can see that is defined as:

> -p, --patch
  Interactively choose hunks of patch between the index and the work tree and add them to the index. This gives the user a chance to review the
  difference before adding modified contents to the index.

> This effectively runs add --interactive, but bypasses the initial command menu and directly jumps to the patch subcommand. See "Interactive mode" for details.

If you read the first line after `git add -p` you'll see git is actually running a diff with `diff --git a/README.md b/README.md`.  In addition to that, you're left a prompt thanks to git running this in `--interactive` mode:

```bash
Stage this hunk [y,n,q,a,d,/,s,e,?]?
```

This prompt is the basis for "Interactive Mode" and the commands are fairly intuitive:

```
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk or any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk or any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
k - leave this hunk undecided, see previous undecided hunk
K - leave this hunk undecided, see previous hunk
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
```

When starting out, I recommended going through your changes with `git add -p`.  I've found debug statements, extra prints and badly described test cases by stepping through code changes with `-p`.  Ok, let's say we're still at our interactive prompt and we're happy with our changes, enter `y` at the prompt.

You should be back at your shell and see that the file has been staged:

```bash
▶ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md
 ```

 From my experience I generally use `y`, `n`, `q` and `s` the most.  Play around with those and enjoy the interactive git commands!

I've aliased this to `g ap` in ~/.gitconfig.
