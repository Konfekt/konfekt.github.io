---
layout: post
title: "Dead-simple dotfiles Management"
date: 2018-11-10
categories: dotfiles version control git linux MacOS
comments: true
---

On a `UNIX` based operating systems, such as `Linux`, `Free BSD` and `MacOS`, it is common practice to put one's `dotfiles`, configuration files whose filename starts with a `.`, under version control (in a `git` repository, for example).

# Aim

Here, we propose to:

1. move all dotfiles of interest from the home folder `~/` to the folder `$XDG_CONFIG_HOME` (which is usually `~/.config`),
1. create symlinks to them (automatically by the shell script [symlink-dotfiles.sh](https://github.com/konfekt/symlink-dotfiles.sh)), and
1. list them (as positives) in an inverted `$XDG_CONFIG_HOME/.gitignore`.

# Steps

1. Move all your dotfiles of interest from `~/` into `$XDG_CONFIG_HOME`
    For example, `mv ~/.bashrc $XDG_CONFIG_HOME/`.
0. Execute the following shell script [symlink-dotfiles.sh](https://github.com/konfekt/symlink-dotfiles.sh) that, for each dotfile (= file whose filename starts with a `.`) `.filename`  in the dotfiles directory `$XDG_CONFIG_HOME`, creates a symbolic link `~/.filename` in the home folder `~/` that points to `$XDG_CONFIG_HOME/.filename`.
    It reads

    ```sh
    dotdir=${XDG_CONFIG_HOME:-${HOME}/.config}
    # Create backup folder named different from any existing one:
    # If the directory already exists, then append '.old'.
    make_backup_dir() {
        if [ -d "$1" ]; then
            make_backup_dir "$1.old"
        else
            mkdir --parents "$1" &&
                echo "$1"
        fi
    }
    backupdir=$(make_backup_dir "${dotdir}")
    dotdir=$(realpath --relative-to="${HOME}" "${dotdir}")
    backupdir=$(realpath --relative-to="${HOME}" "${backupdir}")

    # ignore certain git files and folders
    ignores=(
        .git
        .gitignore
        .gitmodules
    )
    is_ignore() {
        for ignore in ${ignores[*]}; do
            [ "$ignore" = "$1" ] && echo 1 && return
        done
    }

    (
    cd "$HOME" || exit 1
    for file in "${dotdir}"/.[a-zA-Z0-9]*; do
        echo "Found $file ..."

        # keep file name only
        name=$(basename "${file}")

        # skip file if in ignore list
        [ $(is_ignore "$name") = 1 ] && continue

        # If dot file already exists in "$HOME", then move it to backup folder.
        [ -e "${HOME}/${name}" ] &&
            mv --verbose "$HOME"/"${name}" "${backupdir}/$name"

        # create symlink
        ln --verbose --symbolic --relative "$file" "$HOME/$name"
    done

    # remove backup dir if unused
    rmdir --ignore-fail-on-non-empty "${backupdir}"
    )
    ```

0. Create a `$XDG_CONFIG_HOME/.gitignore` that "unignores" all your dotfiles of interest, for example
    
    ```conf
    /*
    !/.gitignore

    !/.bashrc
    !/.bash_profile
    !/.profile

    !/ranger/
    /ranger/*
    !/ranger/rc.conf
    !/ranger/rifle.conf

    !/vim/
    # do not track generated data
    /.vim/spell/
    /.vim/doc/tags
    # do not track plug-in git repos
    /.vim/plugged/
    ```

    This way, all files are ignored except those listed below the first line.
    Note that to ignore all but specific files in a subfolder, the construction given below, for the folder `/ranger` has to be used.
0. Initialize your repository by

    ```sh
    cd $XDG_CONFIG_HOME && git init && git add --all && git commit --message='init'
    ```

# Replication

To use your dotfiles elsewhere,

1. Clone your dotfiles into `$XDG_CONFIG_HOME/` and
1. execute `symlink-dotfiles.sh`.

# Comparison

[Drew Devault's approach](https://drewdevault.com/2019/12/30/dotfiles.html) is similar to this one in being explicit about adding one's dotfiles,

- there by forcefully adding them through `git add --force`,
- here by listing them in an inverted `.gitignore` and symlinking all its files whose name starts with a `.` by `ln --symbolic $XDG_CONFIG_HOME/.* ~/`.

Possibly cleaner than this approach is [an alias of git to a command dotfiles](https://blog.alionet.org/fr/2020-03-29_gerer_ses_dotfiles_avec_git) that stores the `~/.git` folder somewhere else, say in `~/.dotfiles`, to get out of the way in the home folder `~/`;
[Yet Another Dotfiles Manager (yadm)](https://github.com/TheLocehiliosan/yadm/blob/master/yadm) fleshed this out.

