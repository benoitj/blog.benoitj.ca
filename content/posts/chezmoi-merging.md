+++
title = "Chezmoi Merging"
author = ["Benoit Joly"]
date = 2020-10-03T13:13:12-04:00
lastmod = 2020-10-04T11:48:02-04:00
tags = ["100DaysToOffload"]
categories = ["tech"]
draft = false
+++

## Adjusting chezmoi to my workflow {#adjusting-chezmoi-to-my-workflow}

Today, I've been adjusting [chezmoi](https://chezmoi.io) to match my workflow for editing dotfiles.

You may be interested to read my previous post where I go over how to setup and use chezmoi: [Dotfiles management using chezmoi - How I Use Linux Desktop at Work Part5](https://blog.benoitj.ca/2020-06-15-how-i-use-linux-desktop-at-work-part5-dotfiles/)

The workflow I'm seeking:

1.  edit the files in place the same way I edit any files. I don't want to use _chezmoi edit_ on some files, and open files directly in vim or Emacs for other files.
2.  run _chezmoi merge_ on all files changed
3.  configure chezmoi to automatically commit and push

Right now, I need to remember not to run _chezmoi apply_ to prevent loosing changes.


## Issues with default chezmoi behavior {#issues-with-default-chezmoi-behavior}

Implementing my workflow is not easily supported by _chezmoi_:

1.  _chezmoi_ does not support changing the order of files specified when calling the mergetool. kdiff3 merge tool requires a different order to setup base, local, and remote files.
2.  there are no _chezmoi merge --all_ commands to merge all files that changed


## implementing the workflow {#implementing-the-workflow}


### setting up kdiff3 as my mergetool {#setting-up-kdiff3-as-my-mergetool}

_chezmoi_ has an option to specify the mergetool:

```ini
[merge]
  command = kdiff3
  args = [ "-m" ]
```

This has the first issue listed above: kdiff3 is called with files in the wrong order

{{< figure src="/ox-hugo/_20201004_0146262020-10-03_14-08.png" width="130%" >}}

Here is my step process to solve this problem


#### step 1: lets create our own kdiff wrapper script {#step-1-lets-create-our-own-kdiff-wrapper-script}

I could have contributed a change to chezmoi to get this working (and I may will), but I need a solution now.

So I choose to create a script in ~/.local/bin/ called kdiff3merge:

```bash
#!/bin/sh
local="$1"
chezmoi="$2"
base="$3"

kdiff3 -m "$base" "$chezmoi" "$local" -o "$chezmoi"
```

and modify my ~/.config/chezomi/chezmoi.toml:

```ini
[merge]
  command = "kdiff3merge"
```

**note**: ~/.local/bin is in my path

Now, when calling chezmoi merge ~/.zshenv I get a good result:

{{< figure src="/ox-hugo/_20201004_0147112020-10-04_00-40.png" width="130%" >}}

Now I just accept the automatically merged changes by saving the changes and closing kdiff3.

One issue I've found with this is it creates a dot\_zshenv.orig file corresponding to the chezmoi controlled file before the change was applied. I don't need these since I use version control.

```bash
#!/bin/sh
local="$1"
chezmoi="$2"
base="$3"

kdiff3 -m "$base" "$chezmoi" "$local" -o "$chezmoi"

test -f "$chezmoi.orig" && rm -f "$chezmoi.orig"
```

Now orig files are gone.

This is what I needed to fix the first chezmoi limitation I found.


### Run _chezmoi merge_ on all files changed {#run-chezmoi-merge-on-all-files-changed}

Chezmoi has couple of commands that gives information about changes:

1.  chezmoi diff
2.  chezmoi apply -n -v

Both commands are actually too verbose. I could not find a way to just print the list of files to bring in.

Obviously, I ended up creating an alias that does this:

```bash
alias cmm="chezmoi diff -f git | grep -- ^--- | sed -e 's/^--- a\///' | xargs -r chezmoi merge"
```

If we deconstruct this command:

1.  chezmoi diff -f git: prints all changed files, with their changes
2.  grep -- ^---: this one filter in lines which starts with _---_. the -- prefix is to tell grep the --- are not part of the "-" options (like -V)
3.  sed -e 's/^--- a\\///': this command remove from lines found on step 2, the "--- a/" prefix found. at this point, we end up with file path/names only
4.  xargs -r chezmoi merge: runs chezmoi merge on all files found, or run nothing if no files found

Problem solved. I regularly run the _cmm_ alias to bring my changes in chezmoi.


### Commit and push my changes {#commit-and-push-my-changes}

_chezmoi_ supports that off the shelf and can be enabled with the chezmoi.toml:

```ini
[sourceVCS]
  autoCommit = true
  autoPush = true

[merge]
  command = "kdiff3merge"

```


## Conclusion and thoughts {#conclusion-and-thoughts}

CLI tools offer quick ways to extend or customize to fit your habits.

That being said, my scripts may break at any chezmoi release update.

I just cloned the chezmoi repo, and looking to contribute more permanent changes to chezmoi diff and chezmoi merge commands. This will be a good opportunity to learn go.

_This is day 16 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._

<!--more-->
