---
layout: post
title: "Dead-simple dotfiles Management"
date: 2018-11-10
categories: dotfiles version control git linux MacOS
comments: true
---

On a `UNIX` based operating systems, such as `Linux`, `Free BSD` and `MacOS`, it is common practice to put one's `dotfiles`, configuration files whose filename starts with a `.`, under version control (in a `git` repository, for example).

# Aim

Here, we propose to move all dotfiles of interest in `~/` into `$XDG_CONFIG_HOME` (= usually `~/.config`), and create symlinks to them (automatically by the shell script [symlink-dotfiles.sh](https://github.com/konfekt/symlink-dotfiles.sh)).
To track only what is of interest, we propose an inverted `~/.config/.gitignore` that lists positives, that is, files to be tracked.

# Steps

1. Move all your dotfiles of interest from `~/` into `$XDG_CONFIG_HOME`
    For example, `mv ~/.bashrc ~/.config/`.
0. Execute the shell script `symlink-dotfiles.sh` from the repository <https://github.com/konfekt/symlink-dotfiles.sh> that creates

    - for each dotfile (file whose filename starts with a `.`) `~/.config/.filename`  inside the dotfiles directory `$XDG_CONFIG_HOME` (= usually `~/.config`)
    - a symbolic link `~/.file` in the home folder `~/` pointing to `~/.config/.filename`.
0. Create a `~/.config/.gitignore` that reads, for example
    
    ```conf
    /*
    !/.gitignore

    !/.bash_profile
    !/.bashrc
    !/.profile

    !/ranger/
    /ranger/*
    !/ranger/rc.conf
    !/ranger/rifle.conf
    ```

    This way, all files are ignored except those listed below the first line.
    Note that to ignore all but specific files in a subfolder, the construction given below, for the folder `/ranger` has to be used.
0. Initialize your repository by
    ```sh
    cd ~/.config && git init && git add --all && git commit --message='init'
    ```

# Replication

To use your dotfiles elsewhere,

1. Copy `~/.config`
0. Execute `symlink-dotfiles.sh`.

# Comparison

This approach is similar to [this](https://drewdevault.com/2019/12/30/dotfiles.html), in being explicit about adding one's dotfiles,

- there by forcefully adding them through `git add --force`,
- here by listing them in an inverted `.gitignore` and symlinking all its files whose name starts with a `.` by `ln --symbolic ~/.config/.* ~/`.

Possibly cleaner than this approach is [aliasing git to a command dotfiles](https://blog.alionet.org/fr/2020-03-29_gerer_ses_dotfiles_avec_git) that stores the `~/.git` folder somewhere else, say in `~/.dotfiles` to get out of the way in the home folder `~`;
[yadm](https://github.com/TheLocehiliosan/yadm/blob/master/yadm) seems to have fleshed this out.

