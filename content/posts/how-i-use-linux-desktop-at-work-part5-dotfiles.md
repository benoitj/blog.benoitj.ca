+++
title = "Dotfiles management using chezmoi - How I Use Linux Desktop at Work Part5"
author = ["Benoit Joly"]
date = 2020-06-15T10:30:17-04:00
lastmod = 2020-12-18T00:37:08-05:00
tags = ["Linux", "coding", "tools", "vm", "100DaysToOffload"]
categories = ["tech"]
draft = false
+++

Part 5 of the series _How I use Linux desktop at work_ where I describe how I'm building a new Vagrant configuration to use for coding on Windows Host systems.

This post describes how to automate software installation and manage your dotfiles using [chezmoi](https://www.chezmoi.io/).

In this post, I'll also replace openbox with my favorite DM, dwm.

Previous posts:

1.  [part1](https://blog.benoitj.ca/2020-05-29-how-i-use-linux-desktop-at-work-part1-basic-setup/) - Basic setup
2.  [part2](https://blog.benoitj.ca/2020-06-09-how-i-use-linux-desktop-at-work-part2-wm/) - X, and WM
3.  [part3](https://blog.benoitj.ca/2020-06-10-how-i-use-linux-desktop-at-work-part3-guest-additions/) - VirtualBox guest additions
4.  [part4](https://blog.benoitj.ca/2020-06-12-how-i-use-linux-desktop-at-work-part4-dm-with-autologin/) - LightDM and autologin

In the post, I'll cover:

1.  Dotfiles, my dotfiles management history and why I now use [chezmoi](https://www.chezmoi.io/)
2.  Installing [chezmoi](https://www.chezmoi.io/)
3.  [chezmoi](https://www.chezmoi.io/) commands
4.  Install and configure core apps like: dwm, dmenu, st, sxhkd
5.  Integrate with my vagrant setup
6.  Thoughts and what's next


## Dotfiles, my dotfiles management history and why I now use chezmoi {#dotfiles-my-dotfiles-management-history-and-why-i-now-use-chezmoi}


### What are dotfiles? {#what-are-dotfiles}

Lets start with some definitions and descriptions

Dotfiles are files which contains application configuration, settings or preferences. Some dotfiles are edited with a text editor, while some are modified directly in the applications that they serve.

Dotfiles usually starts with a period (.) or are with a folder starting with a period, this is where they take their name.


### Dotfiles location conventions {#dotfiles-location-conventions}

The long time tradition is that dotfiles are stored directly in your home folder (~/). This was quite ok in the old days when you were running couple of apps, but nowadays, this is totally unrealistic. In my system only, I count several thousands of settings files. Most applications nowadays stores their dotfiles under ~/.config.

Appears that the ~/.config is part of the freedesktop.org [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) under the $XDG\_CONFIG\_HOME variable. By default, the specs specifies ~/.config. This is rarely changed (never seen it different).

In your own system, you'll find core utilities dotfiles sitting directly under your home folder, while most applications dotfiles will be under a subfolder of ~/.config.


### My dotfiles management history {#my-dotfiles-management-history}

If you are like me and customize/configure your apps and tools, you will end up with many useful customization you want to carry around with you.

This is where dotfiles management is useful (there is an extensive [github page](https://dotfiles.github.io/) around dotfiles, management tools, and practices).

Personally, I've been managing my dotfiles using various techniques and tools:

1.  started by just zipping them and emailing them around. it helped me for a while
2.  Then put them in a git repo. and manually creating symlinks
3.  Moved to yadm management tool with a bare repo, but did not stick with it. Bare git repos could be risky if you are not cautious. Also could not find out how to manage sensitive configuration files
4.  Back to basic git repo with stow using symlinks. This worked well, but still hard to manage sensitive files.
5.  Replaced stow with makefiles, and envsubst scripts to implement templating. This worked well, but tedious to maintain. Also, if I modified the destination, I would have to remember to integrate the changes also in my git repository.

As you can see, all these solutions are usable, but far from working for me. I was still on a search for a simple dotfiles until i discovered [chezmoi](https://www.chezmoi.io/).

Some key features / benefits that works for me:

-   Like my latest workflow, it install files instead of creating symlink.
-   It supports templates
-   template values can be conditional on the system I install my dotfiles on
-   supports injecting values from commands like password managers
-   Can integrate changes on your target files using merge tools like vimdiff

So now I am porting my dotfiles to [chezmoi](https://www.chezmoi.io/). The remaining of this post will be on my learning journey.

I will share my dotfiles on my [github dotfiles repository](https://github.com/benoitj/dotfiles).


## Installing chezmoi {#installing-chezmoi}

Head over to the [Install | chezmoi.io](https://www.chezmoi.io/docs/install/) page for all options.

In my case, I'm installing on arch so:

```bash
pacman -S chezmoi
```

You can see if chezmoi is properly installed:

```bash
chezmoi --help
```

```text
Manage your dotfiles across multiple diverse machines, securely

Usage:
  chezmoi [command]

Available Commands:
  add              Add an existing file, directory, or symlink to the source state
  apply            Update the destination directory to match the target state
  archive          Write a tar archive of the target state to stdout
  cat              Print the target contents of a file or symlink
  cd               Launch a shell in the source directory
  chattr           Change the attributes of a target in the source state
  completion       Generate shell completion code for the specified shell (bash, fish, or zsh)
  data             Print the template data
  diff             Print the diff between the target state and the destination state
  docs             Print documentation
  doctor           Check your system for potential problems
  dump             Write a dump of the target state to stdout
  edit             Edit the source state of a target
  edit-config      Edit the configuration file
  execute-template Write the result of executing the given template(s) to stdout
  forget           Remove a target from the source state
  git              Run git in the source directory
  help             Print help about a command
  hg               Run mercurial in the source directory
  import           Import a tar archive into the source state
  init             Setup the source directory and update the destination directory to match the target state
  managed          List the managed files in the destination directory
  merge            Perform a three-way merge between the destination state, the source state, and the target state
  purge            Purge all of chezmoi's configuration and data
  remove           Remove a target from the source state and the destination directory
  secret           Interact with a secret manager
  source           Run the source version control system command in the source directory
  source-path      Print the path of a target in the source state
  unmanaged        List the unmanaged files in the destination directory
  update           Pull changes from the source VCS and apply any changes
  verify           Exit with success if the destination state matches the target state, fail otherwise

Flags:
      --color string         colorize diffs (default "auto")
  -c, --config string        config file (default "/home/benoit/.config/chezmoi/chezmoi.toml")
      --debug                write debug logs
  -D, --destination string   destination directory (default "/home/benoit")
  -n, --dry-run              dry run
      --follow               follow symlinks
  -h, --help                 help for chezmoi
      --remove               remove targets
  -S, --source string        source directory (default "/home/benoit/.local/share/chezmoi")
  -v, --verbose              verbose
      --version              version for chezmoi

Use "chezmoi [command] --help" for more information about a command.
```


## chezmoi use cases {#chezmoi-use-cases}


### Getting help {#getting-help}

Chezmoi binary has a built in help. You can get help for any chezmoi commands using the _help_ command followed by the command you want to get help on.

Example:

```bash
chezmoi help help
```

```text
Description:
  Print the help associated with *command*.

Usage:
  chezmoi help [command] [flags]

Flags:
  -h, --help   help for help

Global Flags:
      --color string         colorize diffs (default "auto")
  -c, --config string        config file (default "/home/benoit/.config/chezmoi/chezmoi.toml")
      --debug                write debug logs
  -D, --destination string   destination directory (default "/home/benoit")
  -n, --dry-run              dry run
      --follow               follow symlinks
      --remove               remove targets
  -S, --source string        source directory (default "/home/benoit/.local/share/chezmoi")
  -v, --verbose              verbose
```


### Get started {#get-started}

If you look at the [quickstart](https://www.chezmoi.io/docs/quick-start/), you'll find that all you need to start is:

```bash
chezmoi init
```

then you can add managed files using the _add_ command.

Now lets take ~/.bashrc

```bash
chezmoi add ~/.bashrc
```

This will create a copy if this file under the chezmoi storage folder _~_.local/share/chezmoi/

You can also use the -r option to add files recursively:

```bash
chezmoi add -r ~/.config/newsboat
```


### storage location {#storage-location}

You can _cd_ to the chezmoi storage folder

```bash
chezmoi cd
```

Here you can do any file operations you want.

Note that file names contains encoding for permissions and have a specific meaning for chezmoi.


### Managed files {#managed-files}

From the moment you add a file to chezmoi, it gets copied to the chezmoi local directory and it becomes "managed".

You can find the list of managed files with the command:

```bash
chezmoi managed
```


### version control {#version-control}

Now that we have a file managed, the next thing you may want to do is to version control your dotfiles.

All you need to do is to go to the chezmoi storage folder and initialize the repository.

For git, it would be:

```bash
chezmoi cd
git init
git add .
git commit -m "initial commit with my bash configuration"
```

Chezmoi also has a git command:

```bash
chezmoi git init
chezmoi git add .
chezmoi git commit -m "initial commit with my bash configuration"
```


### Making changes to managed files {#making-changes-to-managed-files}

Any changes made outside of chezmoi's folder will be lost by default.

The best way to edit files is to edit files using chezmoi.

There are really two ways:

1.  call the _edit_ command, like _chezmoi edit ~_.bashrc/
    This command use the EDITOR environment variable to launch your favorite text editor
2.  navigate to the chezmoi's folder and edit them directly there


### Apply changes to managed files {#apply-changes-to-managed-files}

The apply command can install any changes made to managed files on their target (ie: where they are used).

For example, if we changed the ~/.bashrc managed file using the _chezmoi edit ~_.bashrc/ command, we can install the new version using the command:

```bash
chezmoi -v apply
```

**Note**: changes made to the targets are overwritten by the _apply_ command.

If you are unclear what changes will be applied, ensure you use the _--dry-run_ or _-n_ option:

```bash
chezmoi -v -n apply
```

What you can also do is call edit with the -a command to apply after editing. You may also like the -d and -p together with -a to diff and confirm apply.

```bash
chezmoi edit -a -d -p ~/.bashrc
```


### Updating from remote changes {#updating-from-remote-changes}

Here I usually just cd and run git commands.

You can also use the chezmoi update commands which pull changes from git remote and optionally apply changes.

You have choices here.


### Integrate changes made to the target files outside of chezmoi {#integrate-changes-made-to-the-target-files-outside-of-chezmoi}

There are cases where the target file (ex: ~/.bashrc) is modified outside of the chezmoi repo. In this case, if you dont integrate changes in, you will loose them on the next _apply_ command.

You have a couple of options to integrate these changes:

1.  do it manually. edit both files and copy paste changes
2.  a better way is to merge your changes using diff tools.
    chezmoi has the "merge" command to do this

Upon calling _chezmoi merge_ on the specific files edited outside chezmoi, it will start a three-way merge on the file, showing the common ancestor, the value on chezmoi and the local changes. The default tool is vimdiff, but you can define your diff3 tool of choice like meld, or kdiff3.

Here is an example of such diff tool.

On the left, my local changes on ~/.bashrc. On the center, the changes within chezmoi.

{{< figure src="/ox-hugo/_20200616_2225202020-06-16-215937_1366x768_scrot.png" width="150%" >}}

If I want to integrate my local changes to chezmoi, I just need to click the arrow to bring changes from the left pane to the middle one.


### Configuration file {#configuration-file}

The chezmoi configuration file is optional and located in the ~/.config/chezmoi/chezmoi.toml

You can specify the merge tool used by the merge command like here for meld:

```toml
[merge]
  command = "meld"
```


### Templates {#templates}

Early on, I mentionned one reason I selected chezmoi is due to it's templating support.

The easiest way to create a template is by using the auto-templating.

Lets say my ~/.bashrc file is like this:

```bash
export EMAIL="myemail@athome.com"
```

and we want to have different values depending on the system.

So before adding our file to chezmoi, we'll just configure our local email in the chezmoi configuration file:

```toml
[data]
  email="myemail@athome.com"
```

And we add our bashrc file with the autotemplate feature:

```bash
# -T for template
# -a for auto
chezmoi add -T -a ~/.bashrc
```

the resulting template stored in chezmoi would be:

```bash
export EMAIL="{{ .email }}"
```

The _{{ .something }}_ gives you access to values within your configuration file that will be replaced during the _apply_ command execution.

You can see a list of pre-defined variables [here](https://www.chezmoi.io/docs/reference/#template-variables).


### Secrets {#secrets}

Sources for templates can be the configuration file or secret storage tools like bitwarden, gpg, keypassXC, pass, and many more.

See the page [keep data private](https://www.chezmoi.io/docs/how-to/#keep-data-private) for more details.


### Scripts {#scripts}

If you create scripts starting with _run\__ or _run\_once\__, chezmoi will run them at every _apply_ or _once_ unless changed.

For install software, it's better to create _run\_once\__ scripts.


### What else {#what-else}

The items above are the ones I plan to use the most. But chezmoi has much more to offer and the [documentation](https://www.chezmoi.io/docs/reference/) on it's website is quite good.


## Install and configure core apps like: dwm, dmenu, st, sxhkd {#install-and-configure-core-apps-like-dwm-dmenu-st-sxhkd}

I am using run\_once\_core.sh to install dwm, dmenu, st, sxdkd, dunst, status bar, and similar tools.

If you want details, just head over to my dotfiles repository.


## Integrate with my vagrant setup {#integrate-with-my-vagrant-setup}

To get this integrated with my vagrant setup, I need to add chezmoi to the vagrant provision script, and initialize chezmoi from my git repository.


## Thoughts and what's next {#thoughts-and-what-s-next}

This is the last post of the VM development environment using vagrant (at least for now).
The main effort left for me is installing / configuring additional software, also moving some of the vagrant provision script into the chezmoi run\_once script.

Some topics for next posts:

1.  applying patches the git way - or how to patch dwm tools the coder's way
2.  leveraging git history to find hot spot in your source code
3.  test coverage, only half of the answer to quality measure of our automated tests
4.  a series around replacing IntelliJ with Emacs LSP for Java development

_This is day 7 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._

<!--more-->
