+++
title = "Git Merge Conflicts - Git patching and resolving conflict (part 2)"
author = ["Benoit Joly"]
date = 2020-06-28T00:00:00-04:00
lastmod = 2020-06-28T21:44:48-04:00
tags = ["Linux", "coding", "tools", "git", "100DaysToOffload"]
categories = ["tech"]
draft = false
+++

{{< figure src="/ox-hugo/_20200628_190350screenshot.png" >}}

I'm creating this post after seeing numerous people suffering and manually addressing merge conflicts while integrating patches with dwm, st, or dmenu code base.

The most common issues seen:

1.  applying patches on latest from master
2.  using patch instead of git to patch
3.  resolving conflict manually

In my previous [git post](https://blog.benoitj.ca/2020-06-21-git-patching-merging-part1/), I went over how to create and apply patches to a git repository.

In this post, I'll show how conflict can be solved using tools.

**Note**: I'm going to use git, but these techniques are not unique to git.


## Basic setup {#basic-setup}


### setup git merge tool {#setup-git-merge-tool}

In this example, I'll start by using meld diff tool. In a later section, I'll also show different tools.

You first need to install meld.

It can be installed on Linux, windows, OS X. Most distribution package it, or head over to their [website](https://meldmerge.org).

After it's installed, we need to tell git to use this tool as a mergetool:

```bash
git config --global --replace-all merge.tool meld
```

You can then inspect the your ~/.gitconfig and you should see a section like this:

```properties
[merge]
        tool = meld
```


### create a local repository for the experiment {#create-a-local-repository-for-the-experiment}

To do some experiment, I'm creating a local git repository, with 1 file with some text:

```bash
mkdir merge-conflicts-test
cd merge-conflicts-test
git init
cat > file1 <<EOT
original line 1
original line 2
original line 3
EOT
git add file1
git commit -m "initial commit"
```

We have a new repository with file1 containing text:

```text
original line 1
original line 2
original line 3
```

Lets inspect the history by running the command:

```bash
git log --oneline --decorate --relative-date --graph
```

```text
* 8ca4176 (HEAD -> master) initial commit
```


## Trivial conflicts {#trivial-conflicts}

Now lets create a condition for a conflict that is trivial to merge.


### Modify file1 in a branch {#modify-file1-in-a-branch}

We first create a branch and add a line:

```bash
git checkout -b feature/adding-a-line
echo "line 4 from feature branch" >> file1
git add file1
git commit -m "adding some changes at the end of the file"
```

```text
[feature/adding-a-line b24353e] adding some changes at the end of the file
 1 file changed, 1 insertion(+)
```

And we can see what is in file1:

```bash
cat file1
```

```text
original line 1
original line 2
original line 3
line 4 from feature branch
```


### Modify another line on master branch {#modify-another-line-on-master-branch}

Now lets go back to master and add a change to the first line:

```bash
git checkout master
sed -i -e 's/original line 1/new change on line 1/' file1
git add file1
git commit -m "changes to line 1"
```

```text
M	file1
[master 177c8f3] changes to line 1
 1 file changed, 1 insertion(+), 1 deletion(-)
```

And we can see what is in file1:

```bash
cat file1
```

```text
new change on line 1
original line 2
original line 3
```

As you can see, we have 2 commits with different changes to file1.


### Merging {#merging}

We only have a conflict if we want to integrate changes from one branch to another.

Lets do this by merging changes from our feature branch to master:

```bash
git checkout master
git merge feature/adding-a-line
```

```text
Auto-merging file1
Merge made by the 'recursive' strategy.
 file1 | 1 +
 1 file changed, 1 insertion(+)
```

And we can see what is in file1:

```bash
cat file1
```

```text
new change on line 1
original line 2
original line 3
line 4 from feature branch
```

As you can see trivial conflicts are automatically merged.

The history now looks like this:

```bash
git log --oneline --decorate --relative-date --graph
```

```text
*   8061307 (HEAD -> master) Merge branch 'feature/adding-a-line'
|\
| * b24353e (feature/adding-a-line) adding some changes at the end of the file
* | 177c8f3 changes to line 1
|/
* 8ca4176 initial commit
```

What you can read from this history:

-   HEAD points to the latest commit of our master branch, a merge commit
-   the commit "changes to line 1 was done on master"
-   the commit "adding some changes at the end of file" was done on our feature branch
-   the common ancestor of both commits is the "initial commit"


## Non-trivial conflicts {#non-trivial-conflicts}

Now lets create a conflicts that you will have to merge manually.


### Modify file1 on a branch {#modify-file1-on-a-branch}

We will make changes to line 3 from both our feature branch and our master branch:

```bash
git checkout -b feature/change-to-line3
sed -i -e 's/original line 3/line 3 from branch/' file1
git add file1
git commit -m "changing line 3 on our branch"
```

```text
[feature/change-to-line3 2e16635] changing line 3 on our branch
 1 file changed, 1 insertion(+), 1 deletion(-)
```

The history now looks like this:

```bash
git log --oneline --decorate --relative-date --graph
```

```text
* 2e16635 (HEAD -> feature/change-to-line3) changing line 3 on our branch
*   8061307 (master) Merge branch 'feature/adding-a-line'
|\
| * b24353e (feature/adding-a-line) adding some changes at the end of the file
* | 177c8f3 changes to line 1
|/
* 8ca4176 initial commit
```

The commit at the top is on our new feature branch.


### Modify the same line on master {#modify-the-same-line-on-master}

Now lets change line 3 on master:

```bash
git checkout master
sed -i -e 's/original line 3/another change to line 3/' file1
git add file1
git commit -m "changing line 3 again"
```

```text
[master fc9a98c] changing line 3 again
 1 file changed, 1 insertion(+), 1 deletion(-)
```

The history now looks like this:

```bash
git log --oneline --decorate --relative-date --graph
```

```text
* fc9a98c (HEAD -> master) changing line 3 again
*   8061307 Merge branch 'feature/adding-a-line'
|\
| * b24353e (feature/adding-a-line) adding some changes at the end of the file
* | 177c8f3 changes to line 1
|/
* 8ca4176 initial commit
```


### Merging our feature branch into master {#merging-our-feature-branch-into-master}

What happens if we merge our feature branch? We have a conflict that cannot be resolved automatically:

```bash
git merge feature/change-to-line3
```

```text
Auto-merging file1
CONFLICT (content): Merge conflict in file1
Automatic merge failed; fix conflicts and then commit the result.
```

And we can see what is in file1:

```bash
cat file1
```

```text
new change on line 1
original line 2
<<<<<<< HEAD
another change to line 3
=======
line 3 from branch
>>>>>>> feature/change-to-line3
line 4 from feature branch
```

In the file1, you can see changes from both branches.

You could at this point, open your text editor and manually resolve the conflict by keeping the text we want, and removing the conflict markers (<<<<<, >>>>>, `===`).

This is what many people do on YouTube when applying patches and trying to resolve conflicts.
What you may not realize, is we have tools to do this, like meld, vimdiff, kdiff3.

With our previous configuration, all we need is to run the command:

```bash
git mergetool
```

This screen should appear:

{{< figure src="/ox-hugo/_20200625_2047372020-06-25-204447_1363x765_scrot.png" width="150%" >}}

What can you get from this screen:

1.  The middle section is our file1 populated with the common ancestor's value (original line 3)
2.  The left section (called local) contains our change on our current checkout branch
3.  The right section (called remote) contains the change we made on the branch to merge (our feature branch)
4.  You bring the changes you want using the arrow icons.
5.  You can take changes from both local and remote (you can take more than once )
6.  You can edit the middle buffer

I will take both changes, save and close meld.

After all of this is done, you can commit the changes:

```bash
git commit
```

You'll be prompted to keep or change the default commit message.

I'm going to accept the default message.

Voila! Now you know how to resolve conflicts using tools.


## What if you're lost {#what-if-you-re-lost}

You can use the merge --abort option to abort an ongoing merge and then start again.


## Various merge tools {#various-merge-tools}


### kdiff3 {#kdiff3}

Similar tool than meld. GUI based.

You just need to configure your merge.tool git configuration to kdiff3.

Here is a screenshot of the same merge conflict above, but this time with kdiff3:

{{< figure src="/ox-hugo/_20200625_2140312020-06-25-213935_1366x751_scrot.png" width="150%" >}}

Kdiff3 adds a 4th pane, the editing pane.

The base is the common ancestor, the local and remote are like for meld, the changes to the current branch and the branch we want to merge.

Clicking on A/B/C allows you to select which change(s) you want to get. The bottom pane shows you these and also allows you to edit.


### vimdiff {#vimdiff}

Vimdiff is similar to kdiff3 with the 4 panes, except it's console base. You will find your way if your familiar with vim.

Just set the merge.tool git configuration to vimdiff.

Here is the merge conflict as seen with vimdiff:

{{< figure src="/ox-hugo/_20200625_2151152020-06-25-214839_1362x758_scrot.png" width="150%" >}}

Bindings:

move between windows
: C-w C-w

navigate to previous / next conflict
: [c and ]c

find window numbers
: :ls

put the content at point to the destination
: <winnumber>dp

get the content from a buffer
: <winnumber>dg

Window numbers:

1
: local

2
: base

3
: remote

4
: file to edit

Again, the file window can be edited. This is vim, so you can use any vim commands.


## Trivial with non-trivial conflicts {#trivial-with-non-trivial-conflicts}

In case where you have trivial and non-trivial conflicts, tools react differently:

meld
: does not detect trivial from non-trivial

kdiff3
: apply trivial changes automatically. Good most of the time, sometime breaks.

vimdiff
: does not detect trivial from non-trivial


## Thoughts and what's next {#thoughts-and-what-s-next}

The problem of resolving conflicts may not appear to be that big for an example like above, but over many files and many changes, it's the best way to resolve them with consistency and quality.

What I've learned during this post? I still prefer kdiff3 over meld as a merge tool due to automatic trivial conflict resolution of kdiff3. I haven't tried Emacs ediff too, will come when I try replacing IntelliJ with emacs lsp-mode in the future.

In my next post, I'll demo how I organize, apply and merge patches for st using: branches, git apply, and a merge tool.

I hope you learned something :)

---

_This is day 9 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._

<!--more-->
