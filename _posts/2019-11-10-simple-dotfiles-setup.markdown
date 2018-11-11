---
layout: post
title: "A simple dotfiles setup"
date: 2018-11-10
categories: dotfiles version control git linux
comments: true
---

# Intro

On a `UNIX` based operating systems, such as `Linux`, `Free BSD` and `MacOS`, it is common practice to put one's `dotfiles`, configuration files, under version control (in a `git` repository, for example).
In particular, each dotfile (file whose filename starts with a `.`) in `~/` of interest has to be added to the `git` repository.

# Aim

Here, we propose to move all dotfiles of interest in `~/` into `$XDG_CONFIG_HOME` (= usually `~/.config`), and create symlinks to them (automatically by the shell script [symlink-dotfiles.sh](https://github.com/konfekt/symlink-dotfiles.sh)).
To track only what is of interest, we propose an inverted `~/.config/.gitignore` that lists positives, that is, files to be tracked.

# Steps

1. Move all your dotfiles of interest from `~/` into `$XDG_CONFIG_HOME`
    For example, `mv ~/.bashrc ~/.config/`.
0. Execute the shell script `symlink-dotfiles.sh` from the repository <https://github.com/konfekt/symlink-dotfiles.sh> that creates

    - for each dotfile (= file whose filename starts with a `.`) `.filename`  inside the dotfiles directory `$XDG_CONFIG_HOME` (= usually `~/.config`)
    - a symbolic link `~/.filename` in the home folder `~/` that points to `~/.config/.filename`.
0. Create a `~/.config/.gitignore` that reads, for example
    
    ```conf
    /*
    !/.gitignore

    !/.bash_profile
    !/.bashrc
    !/.profile

    # only track rc and rifle.conf inside ranger/
    !/ranger/
    /ranger/*
    !/ranger/rc.conf
    !/ranger/rifle.conf

    !/vim/
    # Generated Data
    /.vim/spell/
    !/.vim/doc/tags
    # Git Repos
    /.vim/autoload/plug.vim
    /.vim/autoload/plug.vim.old
    /.vim/plugged/
    ```

    This way, all files are ignored except those listed below the first line.

    Note that

    - to ignore all but specific files in a subfolder, the construction given below, for the folder `/ranger` has to be used, and
    - albeit we speak of `dotfiles`, of course entire `dotfolders` such as `~/.vim/` can be symlinked to and put under version control.

0. Initialize your repository by
    ```sh
    cd ~/.config && git init && git add --all && git commit --message='init'
    ```

# Replication

To use your dotfiles elsewhere,

1. Copy `~/.config`
0. Execute `symlink-dotfiles.sh`.

