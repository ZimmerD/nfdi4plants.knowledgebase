---
layout: docs
title: Git troubleshooting
date: 2024-01-19
author: 
- name: Dominik Brilhaus
  github: https://github.com/brilator
  orcid: https://orcid.org/0000-0001-9021-3197
add toc: true
add sidebar: _sidebars/mainSidebar.md
---

## About this guide

- This is mostly for data stewards.
- This is not a git tutorial, but rather a small start for troubleshooting.

<a href="./index.html">
  <span class="badge-category">User</span><span class="badge-selected" id="badge-datasteward">Data Steward</span>  
  <span class="badge-category">Mode</span><span class="badge-selected" id="badge-read">Tutorial</span>
</a>
<br>
<br>

## Background

Some reasons, why we now sometimes run into git issues

- Over the past years we asked users (e.g. during trainings) to install git with minium required guidance
- Even though more advanced tools (ARCitect) now bring their own git installation, there might still occur interferences with older installations
- There might also be issues of tools (e.g. ARCitect and ARC commander) or different versions of those tools handling git-related tasks a bit differently or more / less strict (e.g. things like `main` as the default branch)
- The current (versions of) tools were not really built for collaboration with many people on one ARC (at least not with default settings from DataHUB side). So common errors are related to merge conflicts (multiple users changing files) and divergent branches (e.g. between local and remote clones of the ARC).
- Some behaviors are simply very use-case or setup specific and will in any case and even with the best tooling require some stewardship

## Debugging

1. (if required) [Install Git](https://git-scm.com) on user machine

    :bulb: check installation via `git --version` in a fresh command line / terminal / powershell window

2. navigate to the ARC in trouble (via one of many options below)

- On macOS: you can drag&drop the ARC folder from Finder into a terminal
- On macOS: right-click ARC folder-->"Services"--> "New Terminal at Folder"
- On windows: open folder via Explorer; type "cmd" or "powershell" into the address field on top of Explorer
- On linux / macOS terminal: `cd path/to/ARC`

3. try some of the git commands below

:bulb: This is not an exhaustive trouble-shooting list. In most cases git and google are your friends. Most Git error messages (displayed in the command line or inside ARCitect) include helpful commands to solve the problem or can easily be searched for in the internet.

## Your two favorite Git commands: status and log

Whenever your asked for ARC support likely related to a git issue, the first thing you want to explore is the state of the ARC.

### git status

To get a good summary of the ARC including

- the branch you are on
- files that were committed since last commit
- files that were modified, but not committed (tracked)
- typically anything buggy

```bash
git status
```

If everything's clear and committed, this should prompt something like
> Your branch is up to date with ...
> nothing to commit, working tree clean

### git log

Now, to compare the status of the local clone vs. that of the remote (i.e. the DataHUB) with a bit more confidence and wording, use

```bash
git log
```

This displays the commit history (messages) of the ARC reverse-chronologically, i.e. top-most = latest.
So if the top commit message of the local ARC is different from the last commit message displayed in the DataHUB, the ARC is out of sync.

If you like it prettier, remember "a dog"...

```bash
git log --all --decorate --oneline --graph
```

Hit <kbd>q</kbd>to close the log.

## Git configuration

The gitconfig is basically the settings and preferences for your git installation. There are three types of gitconfigs. Depending on the tool (ARCitect, ARC Commander) and operating system (macOS, Linux, Windows), different git settings may be received from different config files.

flag | meaning
------- | -------
--global | current user on that computer
--system | system-wide (all users)
--local | current repository (ARC)

The following command lists all configurations and where they originate (--show-origin) from and what there scope is (--show-scope).

```bash
git config --list --show-origin --show-scope
```

:bulb: The output will be different depending on wether you are inside or outside an ARC (git repository).

In order to only show e.g. the global gitconfig use

```bash
git config --global --list
```

Typical settings to explore and trouble-shoot

- the default branch should be: `init.defaultbranch=main`
- `user.name` and `user.email` should be defined
- if users keep being asked for passwords during sync with the DataHUB, they might not store their credentials via a `credential.helper`.

### Changing git config

Editing the respective gitconfig is ideally done via command line (quick internet search helps).
:bulb: One could edit the file (listed in `git config --list --show-origin`) via a text editor. However, this is rather error-prone.

#### Adapt user name and email

```bash
git config --global user.name "Your Name"
git config --global user.email "Your eMail"
```

#### Set main as default branch

```bash
git config --global init.defaultBranch main
```

## Git remote

For ARCs the remote is the DataHUB. The remote address (ARC url) is stored in the git of the local ARC.
Display the URL, to which the local ARC is connected via

```bash
git remote -v
```

### Adding a remote

A remote is usually added during `arc sync` or via ARCitect.

If above command does not display any remote, you can add one via

```bash
git remote add origin https://git.nfdi4plants.org/<UserName>/<ARCName>
```

### Editing a remote

You can edit a remote via

```bash
git remote set-url origin https://git.nfdi4plants.org/<UserName>/<ARCName>
```

## Branches

As of now, the DataPLANT tools focus on working on a single branch (`main`).
It can still happen that your ARC has multiple branches e.g. by accident (see `git config` --> `init.defaultbranch`) or because some git-affine collaborator knows how to create them.
To display the branches of the local ARC, use

```bash
git branch
```

:bulb: the current branch is marked with an asterisk (*) to the left

If you also want to display branches that exist on the remote (but not locally), use

```bash
git branch --all
```

## Common issues and error messages

### Commit changes before push

Your local ARC is likely out of sync with the remote. Before working on your ARC, update the local clone via one of these

- ARCitect Pull
- `arc sync`
- `git pull` (-> this would also prompt a message if changes need to be merged)

### Access denied

Sometimes you run into permission issues such as

> remote: HTTP Basic: Access denied. The provided password or token is incorrect or your account has 2FA enabled and you must use a personal access token instead of a password.

> fatal: Authentication failed for 'https://git.nfdi4plants.org/UserName/ARCName.git/'

This is due to missing or outdated DataHUB credentials on your computer.
It usually helps to remove existing credentials stored on your computer and retrieve new ones.

#### Authenticate the computer

Option 1: [via ARC Commander](./../ArcCommanderManual/arc_access.html)

Option 2: "by hand"
  1. Login to the [DataHUB](https://git.nfdi4plants.org/)
  2. Create a new [Personal Access Token (PAT)](https://git.nfdi4plants.org/-/profile/personal_access_tokens) with scope `api`
  3. Run a git command (e.g. `arc sync`, `git pull`) to trigger being asked for git credentials
     1. Provide your DataHUB username
     2. Use the token instead of your password

#### Delete stored credentials

If (new) authentication alone does not help, you might need to delete existing tokens or passwords first.

1. Run `git config --get-regexp "credential"` to find out whether and where credentials are stored
2. This typically displays one of the following
   > credential.helper store

   > credential.helper osxkeychain (only on macOS)

3. If `credential.helper store` is displayed, the credentials are typically stored in `~/.git-credentials`, a hidden text file stored in the user's home folder. Edit this file and delete the row(s) containing "git.nfdi4plants.org" (`https://<UserName>:<Token>@git.nfdi4plants.org`).

4. On macOS (if `credential.helper osxkeychain` is displayed) open the app "Keychain Access", search and delete passwords for "git.nfdi4plants.org".

### Dubious ownership

The error `ERROR: GIT: fatal: detected dubious ownership` typically occurs when working on a mounted network drive (Fileshare, File Server, NAS). Very simplified: the user on the computer and the owner of the network drive differ and git tries to safe you from working in a folder you do not own.


You can add the path to the ARC to the list of safe directories via the command

```bash
git config --global --add safe.directory %(prefix)///servername/share/path/to/ARC/
```

You can circumvent this error by adding **all directories** to your list of safe directories via the command

```bash
git config --global --add safe.directory *
```

:warning: This might however pose a safety risk. Please read the details here: https://www.git-scm.com/docs/git-config#Documentation/git-config.txt-safedirectory
