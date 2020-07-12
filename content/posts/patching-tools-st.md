+++
title = "Patching Tools - St"
author = ["Benoit Joly"]
date = 2020-07-12
lastmod = 2020-07-12T12:44:01-04:00
tags = ["Linux", "tools", "git", "100DaysToOffload"]
categories = ["tech"]
draft = false
+++

As part of my renewed dotfiles setup and dev box setup, I'm taking the opportunity to update my core suckless.org tools: st, dwm, dmenu.

I've been using these for some time, and it's great for me to be able to customize tools I use.

In this post, I'll re-create a new ST fork using up to date code base, and new patches that will work for me.

**Note**: The focus will be on branching strategy and resolving merge conflicts. The aim is to show an effective way to leverage git and merge tools to integrate patches. You may want to read my [previous post](https://blog.benoitj.ca/2020-06-28-git-merge-conflicts/) describing how merge conflicts can be resolved using tools.


## initial setup {#initial-setup}

As you may know, git is a distributed version control system that can sync with multiple remotes.

To get started, I usually:

1.  create my own git repository
2.  clone this repository locally (origin)
3.  define an _upstream_ remote
4.  merge the upstream/master branch into my local master branch
5.  update my remote origin

Lets do this:

```bash
git clone git@github.com:benoitj/st.git
cd st
git remote add upstream https://git.suckless.org/st
git fetch upstream
git merge upstream/master
git push
```

The resulting remote setup is:

```bash
git remote -v
```

and my current branch setup:

```bash
git branch -a -vv
```

You can see that my local master branch tracks my github's origin/master branch.


## Patching strategy {#patching-strategy}

For each patch, I will:

1.  create a branch from the commit the patch was created from.
2.  apply the patch. We should not get conflicts here.
3.  merge the patch to master branch (may need conflict resolution. Read my [previous post](https://blog.benoitj.ca/2020-06-28-git-merge-conflicts/))

How do we know how to create the branch?

In the case of most patches, the filename indicate some context. In the following example, we will apply the patch _st-xresources-20200604-9ba7ecf.diff_. From  the file name, we understand the patch was created June 4th from the commit hash 9ba7ecf.

Now the trick is to create our branch from that commit. Reading the git documentation, you can learn this is done with the syntax: git branch <branch name> [commit hash]

Sometimes the commit hash does not match the history, in this case, use git log and find a commit hash close to the patch creation date.


## Patching ST {#patching-st}

Lets do this and apply the patch:

```bash
mkdir patches
wget -P patches https://st.suckless.org/patches/xresources/st-xresources-20200604-9ba7ecf.diff
git branch patches/xresources 9ba7ecf
git checkout patches/xresources
git apply < patches/st-xresources-20200604-9ba7ecf.diff
```

And we can see what has changed:

```bash
git diff
```

and lets confirm it worked:

```bash
make clean all
./st
```

Everything's good, now is a good time to commit our changes:

```bash
git add x.c config.def.h patches
git commit -m "applying patch st-xresources-20200604-9ba7ecf.diff"
```

And push:

```bash
git push --set-upstream origin patches/xresources
```

and merge the branch to master:

```bash
git checkout master
git merge patches/xresources
```

I will do the same for the following patches: externalpipe and scrollback:

```bash
wget -P patches https://st.suckless.org/patches/scrollback/st-scrollback-20200419-72e3f6c.diff
git branch patches/scrollback 72e3f6c
git checkout patches/scrollback
git apply < patches/st-scrollback-20200419-72e3f6c.diff
make clean all
```

```bash
git add st.c st.h config.def.h patches
git commit -m "applying patch st-scrollback-20200419-72e3f6c.diff"
git push --set-upstream origin patches/scrollback
```

```bash
git checkout master
git merge patches/scrollback
```

If I build master, I have both scrollback and xresources patches included:

```bash
rm config.h
make clean all
```


## Merge conflicts {#merge-conflicts}

You may get couple of merge conflicts around keybindings, configuration, and similar.

They are usually really simple to solve using a merge tool as described in this [previous post](https://blog.benoitj.ca/2020-06-28-git-merge-conflicts/).

I had no conflict patching st, but had some when patching dmenu. Here are some examples:


### Example of applying borderoption patch on top of border patch {#example-of-applying-borderoption-patch-on-top-of-border-patch}

```bash
git checkout master
git merge patches/border
git merge patches/borderoption
git mergetool
git commit
```

Example of such conflict when merging border and borderoption on dmenu master branch:

{{< figure src="/ox-hugo/_20200712_120511screenshot.png" width="150%" >}}

In this case, I will select the _remote_ (the borderoption branch) since it makes the attribute mutable.


### Example of applying the dmenu fuzzymatch patch on top of center patch {#example-of-applying-the-dmenu-fuzzymatch-patch-on-top-of-center-patch}

```bash
git checkout master
git merge patches/center
git merge patches/fuzzymatch
git mergetool
git commit
```

{{< figure src="/ox-hugo/_20200712_120910screenshot.png" width="150%" >}}

In this case, we need to take both the _local_ (center patch) and the _remote_ (fuzzymatch patch), so I select C and B to get this result:

{{< figure src="/ox-hugo/_20200712_121232screenshot.png" width="150%" >}}


## Thoughts and what's next {#thoughts-and-what-s-next}

I hope I was able to show how simpler patching tools can be when using a branching strategy, branching from the right version, and using tools to resolve conflicts.

Yes you can do all this manually on the same branch, but it's takes more time and it is much more error prone. Tools are better than us to identify patterns and repeat tasks.

Have fun patching! :)

---

_This is day 10 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._

<!--more-->
