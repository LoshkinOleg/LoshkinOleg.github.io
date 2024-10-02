---
title: "Ubuntu Directories Cheatsheet"
layout: cheatsheet
categories: "cheatsheets"
permalink: /:categories/:title
---

# User's Stuff
- /home : Folder containing the workspaces of non-system users. TODO
- /opt : Installation directory for software not managed by package managers. That's where rebelious software goes, that doesn't like to play by the conventions. It's for "self-contained" programs. This folder you should be managing manually. Prefer using /home/your_name/opt instead.
- /home/your_name/opt: User-specific /opt equivalent. Good place for "compile it yourself and install" type of software. Manage manually.
- /usr : Read-only stuff, that's where most software is installed. Used by package managers. Usually for packages provided by the OS. Don't mess with it directly, go through the package manager instead.
- /usr/local : Read-only stuff, that's where polite software installs itself. Can be used by package managers or not. Again use the package manager to edit this directory if you can. Otherwise use it for installing conventionally separated software (bin, lib, share) Protected from being overwritten upon OS update.

# Configuration
- /etc : Location of system-wide config files, for all users. That's where "environment" file is located, where PATH is defined. Don't mess with unless you know what you do.

## Don't mess with stuff beyond this line
-----

# Executables
- /bin : Executables of system terminal commands like <em>ls</em> . Don't mess with it.
- /sbin : Executables of the system terminal used by the superuser. Don't mess with it.
- /lib : Location of system-specific dynamic libs, not for user installed software. Don't mess with it.

# Memory
- /dev : <strong>Internal</strong> devices the OS has access to, like your hard drives or GPU. Don't mess with it.
- /media : Mounting point for <strong>external</strong> devices, like USB drives or CD's. Don't mess with it.
- /mnt : Same as above, but for network stuff? Don't mess with it.
- /sys : Virtual filesystem, used to see what the kernel sees. Don't mess with it.
- /proc : Virtual filesystem, used by the OS to communicate with processes? Don't mess with it.

# Misc
- /root : Don't.
- /boot : Files for booting the system. Don't mess with it.
- /run : Temporary directory for startup stuff. Don't mess with it.
- /tmp : Temporary files. Feel free to mess with it.
- /srv : For files transfer stuff, like FTP. Don't mess with it.
- /var : Directory for persistent variable data like logs. Don't mess with it.