+++
title = "Git patching and resolving conflict (part 1)"
author = ["Benoit Joly"]
date = 2020-06-21
lastmod = 2020-06-21T22:55:41-04:00
tags = ["Linux", "coding", "tools", "git", "100DaysToOffload"]
categories = ["tech"]
draft = false
+++

I'm creating this post after seeing numerous people suffering and manually addressing merge conflicts while integrating patches with dwm, st, or dmenu code base.

The most common issues seen:

1.  applying patches on latest from master
2.  using patch instead of git to patch
3.  resolving conflict manually

Applying patches and resolving conflict is the land of version control system like git and we have many techniques and tools to make it easy.

This is not specific to suckless.org tools, but I'll use some example patches to show how to use tools to automate as much as we can.

**Note**: I'm going to use git, but these techniques are not unique to git.


## Definitions {#definitions}

These terms may be oversimplified, but it's done to keep things simple.


### A commit {#a-commit}

A registered set of files that changed.

In git, a commit has a unique identifier (a hash code).
Git only track changes made to files, so you can't register a folder in a commit.

A commit has a parent commit (except for the initial commit) and cumulative changes are actually a series of commits, each commits pointing to it's parent.

{{< figure src="/ox-hugo/commits.png" >}}


### branch {#branch}

A way to parallelized software development. You make changes (commits) to a branch, and you create other branches to separate the work.

In git, the primary development branch is by default called master.

Git branches are actually pointers to the latest commit of the branch. Creating new commits on the branch moves the pointer to the new commit.

{{< figure src="/ox-hugo/master.png" >}}

Now imagine we want to do some development work while keeping our primary branch stable, we would create another branch:

{{< figure src="/ox-hugo/feature.png" >}}

Now development continues and we have diverging changes but it's not a problem, we are on different branches.

{{< figure src="/ox-hugo/feature-diverge.png" >}}


### Fully decentralized {#fully-decentralized}

Something many people don't realize is that git model is decentralized by design. We can have multiple equivalent copies of the same repository. It happens that most people use centralized systems to share code, but this is not mandatory. The whole central system could go and you can still share code and haven't lost code.


### Merge {#merge}

Now that we are done with our feature, we want to integrate it into master.

We use git merge to merge our _myfeature_ branch to master:

{{< figure src="/ox-hugo/feature-merge.png" >}}


### Conflicts {#conflicts}

In the merge scenario, all is good if the diverging changes did not touch the same file. If we touched the same file, we end up in conflict. Git and merge tools would resolve trivial conflicts automatically (when we did not change the same lines).

We do need to manually merge conflicts when changes occurs on the same lines.


### Manual conflict resolution with 3 panes diff tools {#manual-conflict-resolution-with-3-panes-diff-tools}

Now, this is manual in the sense that we need to make a decision, but we do not need to copy paste lines, edit the files manually. We have tools for this folks!

There are many tools that we can use to merge conflicts:

1.  vimdiff
2.  emacs ediff
3.  kdiff3
4.  meld
5.  Your IDE might have one
6.  many more

a 3 panes diff tool usually look like this:

```text
|------+-------+--------|
| base | local | remote |
|------+-------+--------|
|    editing buffer     |
|------+-------+--------|
```

a 3 panes diff tool usually look like this:

```text
|------+---------------------+--------|
| base | local / edit buffer | remote |
|------+---------------------+--------|
```


## Patches {#patches}

A patch is a diff file (added lines/files, removed lines/files) taken at a point in time relative to a specific version.

One way to share changes is by sending patches. git even has an email client to send patches.

{{< figure src="/ox-hugo/patch.png" >}}

Now if i'm on the latest of myfeature, I can create a patch against master by typing the command:

```bash
git format-patch master --stdout > myfeature.patch
```

Now this patch what taken from the latest commit of master, so it is good practice to add the commit id to the patch. This way people know how to apply the patch.

{{< figure src="/ox-hugo/before-apply.png" >}}


## Applying patches {#applying-patches}

Now lets say, time has passed, and master has additional changes.

The common mistakes is trying to apply the patch on the latest master branch.

This may work, but as time pass, you may end up with issues.

The right way to apply this patch is to create a new branch from the commit this patch was taken from, and then merge this branch to master.

{{< figure src="/ox-hugo/branch-apply.png" >}}


### Thoughts and what's next {#thoughts-and-what-s-next}

Time to send this. I realize this problem needs to be split in couple of manageable pieces.

This post was to set ground on git terminology, and patching techniques and address one of the problems often seen when patching: patching from the right place in the history.

I see two possible follow-up posts:

1.  merge conflict resolution
2.  real life example using dwm, st, or dmenu

---

_This is day 8 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._

<!--more-->
