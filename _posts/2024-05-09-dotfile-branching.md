---
layout: post
title: "Git Dotfile Branching"
date: 2024-05-09
categories: Git Dotfiles Branches Work Personal Machine
comments: true
---

Working with multiple computers, like a work and a personal laptop, often involves differences in operating permissions, applications, systems, ...
For example:

- While symbolic linking of files is allowed on Linux, not so on the administered Windows work machine.
- Some software might work fine on Ubuntu but not so on OpenSUSE, for example, by different versions of dependent libraries, like `glibc`.
- Work and personal laptop use different directories for storing documents,    requiring conditional configurations.

To manage these differences, maintain separate Git branches for each machine's specific configurations.
My current strategy builds onto [Greogry Anders' blog post](https://gpanders.com/blog/managing-dotfiles-with-git/):

# Centralization

Maintain as many common dotfiles as possible in the `main` branch (formerly `master`); ensuring that any universal changes benefit all setups.
Adding a folder for tools predominantly used on one machine but less on others does little harm, but removes friction.
For example, the `Zsh` and `Bash` profile, as well as `Autohotkey` scripts (for setting up key combos in Windows, among others), can be all kept in the `main` branch.

# Change Management by Rebasing Commits on top of the Main Branch

Commits on branches other than the master branch are always kept on top of those of the master branch, ensured by `git rebase --onto` (as is common practice for a long running side branch):

- When you need to integrate updates from the `main` branch into a side branch like `work`, use the command `git rebase --onto main HEAD~123` where `123` is the number of commits since `work` diverged from `main`.
    This command rewrites the history of the `work` branch to include the latest changes from `main` without duplicating commits.

- Conversely, if changes were made directly to the side branch `work` and need to be reflected in the `main` branch, then use `git branch --force main HEAD~123` where `123` again designates the number of commits since `work` diverged from `main`.

In essence, that summarizes the process.
To replicate your changes of a branch, say `work`, onto different computers 

- pull them with `git pull --force origin work:work`, and
- push them with `git push --forward-with-lease origin work:work`.

