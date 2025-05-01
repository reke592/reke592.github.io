---
layout: post
title: Linux Basic (Ubuntu)
date: 2025-05-01 19:11:00 +0800
categories: devops
---

### Introduction

The primary reasons why a Windows user struggle with Linux is because of the radically different directory structure and file permissions. Here we will tackle the basic to provide the minimal understanding.

### Directory Structure

The Linux filesystem organizes components logically by purpose, unlike Windows, which often centralizes everything under `C:\`. Here’s a breakdown of key directories and their Windows equivalents:

#### Windows:

- C:\\Users\\`username`
- C:\\Program Files\\MySQL

#### Linux:

- /home/`username`
- system components are distributed into multiple directories
  - `/bin` -> `/usr/bin` - system tools, builtin or installed by the package manager. <u>It may require `sudo` privilege depending on how the execution interacts with the file system.</u>
    - `cd, cp, ls, mount` bultins
    - `mysql` client
    - `docker` cli
  - `/sbin` -> `/usr/sbin` - system administration tools. <u>require `sudo` privilege to run</u>.
    - `mysqld` server
    - `dockerd` server
    - `nginx` server
  - `/usr/local/bin` - shared tools <u>downloaded or created by a user. require `sudo` privilege depending on how the execution interacts with the file system.</u>.
    - `docker-compose`
  - `/lib` -> `/usr/lib` - library dependencies of a package
    - `/usr/lib/mysql`
    - `/usr/lib/apache2`
  - `/usr/share` - e.g. documentations, fonts, locale, configuration templates.
    - `/usr/share/nginx/modules-available`
    - `/usr/share/python3`
  - `/etc` - package configuration files
    - `/etc/mysql`
    - `/etc/nginx`
  - `/opt` - standard directory for <u>third-party packages not managed by the system's package manager</u>. These packages are commonly installed by extracting a .tar.gz file or by running a setup scripts.
    - `/opt/google/chrome`
    - `/opt/jetbrains`
    - `/opt/zoom`
  - `/snap` - directory for <u>third-party packages managed by Canonical, self-contained that updates automatically</u>
    - `/snap/docker/current -> 1234`
    - `/snap/docker/1234/bin`
  - `/var/lib` - variable dependencies of application while running
    - `/var/lib/mysql` database files
  - `/var/log` - variable data storage application logs while running,
    - `/var/log/mysql`
    - `/var/log/nginx`

### Package Managers

e.g.

#### Ubuntu:

- `apt` - package manager for Ubuntu, install apps from official Debian/Ubuntu Repository.
- `snap` - package manager for Linux, managed by Canonical, self-contained apps running in sandboxed environment.

```
sudo apt update
sudo apt search nginx
sudo apt install nginx
```

#### Windows:

- `winget` - package manager for Windows. Microsoft Store apps are sandboxed `(similar to snap)`

```
winget search nginx
winget install nginxinc.nginx
```

#### Performance Considerations

Sandboxed apps prioritize `security and isolation` but often sacrifice performance.

- Slower startup times (due to virtualization)
- Higher resource usage
- Restricted system access

Recommendations:

- Use native packages (apt) for `servers`/performance-critical apps
- Use sandboxed apps only when necessary (e.g. proprietary `desktop` software)

### Top 10 commands and options

- `ls` - list all files in current directory
  - options:
    - `-l` - show file permissions
    - `-a` - show hidden files
    - `-h` - show with human readable file size
    - `-i` - show the file INode ID.
  - example:
    - `ls -lh`
- `cd` - change directory
  - example:
    - `cd /target/path` - navigate to target path
    - `cd ~` - shortcut to navigate the current user home directory
- `cp` - copy
  - options:
    - `-r` - recursive
    - `-a` - preserve file permission
  - example:
    - `cp file1.txt file1.bak` - copy file1.txt to new file1.bak
    - `cp source/* target` - copy all files in source directory to target directory
    - `cp -ra source target` - copy the source directory to target directory, preserve the file permissions.
- `mv` - move
  - example:
    - `mv file1.txt file1.bak` - rename file1.txt to file1.bak
    - `mv source/* target/` - move all files in source directory to target directory
    - `mv source target/` - move the source directory in target directory
- `rm` - remove
  - options:
    - `-r` - recursive
    - `-f` - force
  - example:
    - `rm file1.txt` - deletes the file1.txt
    - `rm source/*` - delete all files in source directory
    - `rm -r source` - deletes the source directory
- `mkdir` - create new directory
  - options:
    - `-p` - create a folder including sub-directories in given full path
  - example:
    - `mkdir target` - create new directory name target
    - `mkdir -p folder1/folder2/target` - create new directories folder1, folder2 and the target
- `grep` - searches for PATTERNS in each FILE
  - example:
    - `grep "pattern" file1.txt` - display all lines in file1.txt that matches the pattern
- `man` - read the manual page of specific command
  - example:
    - `man ls` - read the user manual of the command `grep`
    - `man ls | grep "all"` - read the user manual of `ls` and look for string pattern `all`
- `lsblk` - list block (storage) devices
- `mount` - mount block to directory
  - example:
    - `mount /dev/sda1 targetDirectory` - mount block device name sda1 to target directory
    - `umount /dev/sda1` or `umount targetDirectory` - to unmount the device name sda1

To run a command as super user prefix it with `sudo`. e.g. `sudo cp -ra source target`

### Permissions

Flags:

- 4 - **`r`**ead
- 2 - **`w`**rite
- 1 - e**`x`**ecute
- 0 - no permission

To change the permission of a file or directory we use the `chmod` tool.

Example:

```sh
# Run the commands below to create a file and display the permission
touch filename.txt
ls -l

-rwxrwxrwx 1 username username     0 May  1 19:30 filename.txt
```

```sh
# Run the commands below to change the file permissions
sudo chmod 664 filename.txt
ls -l

-rw-rw-r-- 1 username username     0 May  1 19:30 filename.txt
```

The file permission string is divided into 4 parts (from left to right).

- position 1 filetype `-`
  - `-` reglar file
  - `d` directory
  - `s` socket file
- position 2-4 user access `rw-`
  - `r` read (4)
  - `w` write (2)
  - `x` execute (1)
- position 5-7 group access `rw-`
- position 8-10 public access `rw-`

Adding the Flag value will combine the permissions.

- `7` = `rwx`
- `6` = `rw-`
- `5` = `r-x`
- `3` = `-wx`
- `0` = `---`

### Special Permissions

Flags:

- 4 - User permission
  - effect on executable files: Runs with the permissions of the file owner (rather than the user executing it)
- 2 - Group permission
  - effect on executable files: Runs with the permissions of the file's group
  - effect on directories: New files created inside inherit the directory's group ownership. (useful for shared folders)
- 1 - Sticky flag
  - effect on directories: only the file owner, directory owner or root can delete or rename files inside (even if others have write permissions)

To assign special permissions we include the special flag before the file permission flags in `chmod`

```sh
sudo chmod 4751 executable
sudo chmod 6770 directory
ls -l

-rwsr-x--x  1 username username        0 May  1 12:13 executable*
drwsrws---  1 username username        0 May  1 12:13 directory/
```

Or use the short term

```sh
sudo chmod u+s executable
sudo chmod g+s directory
ls -l

-rwsr-x--x  1 username username        0 May  1 12:13 executable*
drwsrws---  1 username username        0 May  1 12:13 directory/
```

Assign Sticky flag to directory

```sh
sudo chmod 1777 directory
# or
sudo chmod +t directory
ls -l

drwxrwxrwt  1 username username        0 May  1 12:13 directory/
```

Capital leter `S` indicates invalid special permission. From the below example a special permission from file owner or group was given but the file owner or file group has no permission to execute.

```sh
sudo chmod 6701 executable
sudo chmod 6665 directory
ls -l

-rws--S--x  1 username username        0 May  1 12:13 executable*
-rwSrwSr-x  1 username username        0 May  1 12:13 directory/
```

### Removing Special Permissions

We can remove the special bits by using `-` instead of `+` in `chmod` command.

```sh
sudo chmod u-s executable
sudo chmod g-s directory
sudo chmod -t directory
```

<center>- end -</center>
```
