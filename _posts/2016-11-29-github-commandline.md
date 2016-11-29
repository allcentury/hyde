---
layout: post
title: Jump to github from your shell
---

I've been going through the ['Learning the bash Shell: Unix Shell Programming' by Cameron Newham](https://www.amazon.com/Learning-bash-Shell-Programming-Nutshell/dp/0596009658) and ['Classic Shell Scripting' by Arnold Robbins](https://www.amazon.com/Classic-Shell-Scripting-Arnold-Robbins/dp/0596005954/) over the last few months.  I've learned a lot about the \*nix way and I'm confident there is a tremendous amount more to learn.

The books encourage you to think about repetitive tasks you make on your day-to-day and try to automate them. I find myself constantly checking in new code locally with git and then going to github to either find the issue number I'm working on, or to reference a pull request by another team member.  That means I'm sitting at my bash prompt, I leave to go to chrome, I go to chromes address bar and hope I remember the repository name.  Wait, was this on github or bitbucket?  Is the local directory the same as the repository name on github?  That's when I realized I should try to automate this.

This is not rocket science and I'm merely sharing because it's saving me a lot of time, especially for it being so simple.  Hopefully it helps others as well.

Here's the bash script, go ahead and copy this into where you have user-wide scripts available.  I like to put mine in `~/scripts`, go ahead and name this `remote`.

```bash
#! /bin/bash

endpoint=$1

get_ext() {
  case $1 in
    *github*)
      case $endpoint in
        pr)
          echo "/pulls";;
        issues)
          echo "/issues";;
        *)
          echo ;;
      esac;;
    *bitbucket*)
      case $endpoint in
        pr)
          echo "/pull-requests";;
        issues)
          # issues aren't enabled by default for bb repos
          # so this might not always work
          echo "/issues";;
        *)
          echo ;;
      esac;;
  esac
}

open_site() {
  ext=$(get_ext $1)
  url=""
  if [[ $remote == *@* ]]; then
    # extract path from ssh
    name=$(echo "$remote" | sed  "s/^.*:\(.*\)\.git/\1/g")
    url="$1/${name}${ext}"
  else
    # extract uri from http
    name=$(echo "$remote" | sed "s/^.*: \(.*\).git/\1/g")
    url="${name}${ext}"
  fi
  open "$url"
}

# get remote from origin
remote=$(git remote show -n origin | grep Fetch.URL)

case $remote in
  *github*)
    open_site "https://github.com";;
  *bitbucket*)
    open_site "https://bitbucket.org";;
esac
```

You'll want to alias that script in your `~/.bashrc` or `~/.zshrc` like this:

```bash
alias github="~/scripts/remote"
alias bitbucket="~/scripts/remote"
alias remote="~/scripts/remote"
```

For whatever reason my brain thinks everything is on github so I generally just use the first alias from the command line. Go ahead and re-source your shell via `. ~/bashrc` (or zshrc depending on what you use).

Go to a directory with a git repository:

```
$ github
$ github pr
$ github issues
```

There you have it!  Right now the script assumes the `origin` remote has the repo information (ie whether it lives on github or bitbucket).  That is 99% true for my projects but it might not be for yours.
