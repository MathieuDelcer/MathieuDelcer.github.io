---
layout: post
title: Linux LFCS Certification Study Notes
date: 01-12-2024
categories: [study-notes]
tag: [linux, certification]
---

## Context
This documentation covers my Linux System Administration revision notes. These are also my study notes taken during the preparation for the LFCS (Linux Foundation Certified System Administrator) certification exam.

## Help and Manuals

**man**

`man man`

`man 2 man` #To open chapter 2

`/EXAMPLES` #Once in the documentation to find examples

**Getting help on a command**

`ls --help`

**whatis**

`whatis python` #Gives a description of the command

**apropos**

`sudo mandb` #To make apropos work

`apropos director` #To search for commands related to directories

**tldr**

`apt yum install tldr -y`

`tldr -u` #Updates the database

`tldr find` #Gives a summary for the find command

**which**

binary path

`which python` #Returns the full path of shell commands, e.g.: # /usr/bin/python

**whereis**

`whereis python` #Gives more information, (location of binaries, source files, and manual pages)

## Directories

**Navigation**

`cd -` #Return to the previous directory

`cd` # Return to the home

**Important Directories**

```
Directory	Description
/	    This is the root of the system partition.
/bin	Stores essential executables and binaries
/boot	Stores Linux boot files
/dev	Files related to devices
/etc	Linux and application configuration files
/home	User directories
/lib	Shared libraries for the operation of the OS and applications
/lost+found	Fragments of files recovered by fsck
/media	Contains mount points for removable media
/mnt	Directories used to temporarily mount a file system (floppy disk, CD-ROM, etc.).
/opt	Applications installed from a source other than the distribution's package system
/proc	Virtual directory with system information (system status, Linux kernel, etc.) based on procfs (process file system)
/root	The personal folder of the root user
/sbin	System executables and binaries
/srv	Files related to services
/tmp	Temporary folder
/usr	User application directory
/var	Variable data frequently written
```

## User Environment

**Env Files**

`/etc/profile` #This is the generic file that is processed by all users upon login.

`/etc/bashrc` #This file is processed when subshells are started.

`~/.bash_profile` #In this file, user-specific login shell variables can be defined.

`~/.bashrc` #In this user-specific file, subshell variables can be defined

**Users/Groups Files**

`/etc/shadow` #Users' encrypted passwords and other information about the passwords

`/etc/passwd` #User records

`/etc/group` #Group records

**Configuration**

*The .profile file is loaded at session startup, while the .bashrc file is loaded each time an interactive Bash terminal is opened. Both are used to configure the user environment, but .profile is more general, whereas .bashrc is specific to the Bash shell.*

`vim ~/.profile` #You can add variables here that will be set automatically at login

`sudo vim ~/.bashrc` #Script executed each time the terminal is opened. Useful for adding aliases, redundant commands, etc.

`sudo vim /etc/profile.d/lastlogin.sh` #Create a script that will run at every login for all users. No need for a Shebang in this folder. Example:

```sh
echo "Your last login was at : " >> $HOME/lastlogin
date >> $HOME/lastlogin
```

## Environment Variables

**Display environment variables**

`env` or `printenv`

`echo $LANG` #Display a variable

`echo $PATH` #PATH Variable 

**Add an environment variable**

`test=test`

`echo $test`

**Add a variable for all users**

`sudo vim /etc/environment` #Affects all users (system-wide)

`source /etc/environment` #Reload the file without rebooting

## Package manager

### Search, Install, Remove (apt, dpkg, dnf)

**On Debian-based distros (Ubuntu):**

`sudo apt update && sudo apt install <package>` #Update the package list and install the package

`sudo apt remove <package>` #Remove the package

`sudo apt autoremove` #Removes dependencies of packages that are no longer used

`apt show nginx`  #Shows what the package does

`apt search nginx` #Returns packages that contain "nginx" in the name or description

`apt search --names-only nginx` #Returns packages that contain "nginx" only in the name

**dpkg = low-level tool for managing packages**

`dpkg --listfiles nginx` #List the files of the nginx package

`dpkg --search /usr/sbin/nginx` #Find the package that uses the file

**For RH-based distributions (CentOS), apt=dnf and dpkg=rpm**

`dnf search nginx`

`rpm --query --file /usr/bin/tldr`

### Configure the repository

`sudo vim /etc/sources.list` #Contains the repository URLs

**Add a 3rd party repo while verifying authenticity**

Retrieve the public key and add it (ensures the package is original and not modified by a third party)

`curl "https://download.docker.com/linux/ubuntu/gpg" -o docker.key`

`gpg --dearmor docker.key`

`sudo mv docker.key.gpg /etc/apt/keyrings/`

`sudo vim /etc/apt/sources.list.d/docker.list` :

```
deb [signed-by=/etc/apt/keyrings/docker.key.gpg] https://download.docker.com/linux/ubuntu jammy stable
```

`sudo apt update` #Update to take changes into account

### Personal Package Archives (PPA)

*Personal Package Archives (PPA) allow you to upload Ubuntu source packages to be built and published as an apt repository by Launchpad*

`sudo add-apt-repository ppa:graphics-driver/ppa` #Use the same command with the --remove flag to remove it

This command handles key addition, source modification, etc.

### Install software from source files

`git clone <url>`

Inside the folder, check the README. It usually contains installation steps (e.g., installing dependencies...)

`./script.sh` 

`./configure`

`sudo make install` #Compile and install in /usr/local/bin

`./application` #Now you can launch the application

## Links

**Hard links**
Hard link = mirror copy. Shares the same inode. If the original is deleted, the hard link remains. Only possible for files, not directories.

`ln original_file hard_link` #Create the hard link

`stat hard_link` #See the number of links

`unlink symlink_name` #Delete the link

`rm symlink_name` #Delete the link

**Soft links**

Soft link = "Shortcut." If the original is deleted, the soft link no longer works. Works with directories.

`ln -s original_file soft_link`

## Ownership

You can view a file's group with `ls -l`

**Change a file's group**

`chgrp groupname my_file.jpg` #You can only choose a group you belong to

**Change the owner**

`sudo chown username my_file.jpg` #Requires root privileges, so we use sudo to get them temporarily

**Set the group as the owner of a directory**

`sudo chown :group01 /var/test/`

**Change both owner and group at the same time**

`sudo chown username:groupname my_file.jpg`

## User management

**Add a user**

Low-level command

`sudo useradd -m user01` #Creates the user and the associated group. `-m` adds the user directory in `/home`

High-level command, more user-friendly

`sudo adduser user01`

**Add a system user**

`sudo useradd --system sysacc`

**Add/Modify a user's password**

`sudo passwd user01`

**Lock/Unlock a user's password**

`sudo passwd -l employee2` #Lock password

`sudo passwd -u employee2` #Unlock password

**Delete a user**

`userdel user01` #Deletes the user and the associated group

`userdel -r user01` #Removes the associated directory as well

**Modify user parameters**

`sudo usermod --home /home/otherdirectory --move-home john` #Change home directory

`sudo usermod --login jane john` #Change username login

**Lock a user account**

`sudo usermod --lock john`

`sudo usermod --unlock john`

**Set account expiration**

`sudo usermod --expiredate 2025-12-20 john` #Account expires on the specified date - login is no longer possible

`sudo usermod --expiredate "" john` #Expires immediately

**Set password expiration**

`sudo chage --lastday 0 john` #User must change password at next login

`sudo chage --lastday -1 john` #Cancel password expiration

`sudo chage --maxdays 30 john` #User must change password every 30 days

`sudo chage --maxdays -1 john` #User's password never expires

`sudo chage --list john` #View password information for user

**Skel directory**

Contains files that will be copied into a new user's home directory

`ls /etc/skel`

`vim /etc/skel/README` #Example usage

**Display user information**

`id user01`

**View user attributes**

`ls /etc/passwd`
```
Name:Password:UserID:PrimaryGroup:Gecos:HomeDirectory:Shell
```

**Resource Limits**

`sudo vim /etc/security/limits.conf`

Example to limit the number of processes a user can run simultaneously (`@` for groups, e.g., `@developers`):
```
john - nproc 3
```

`ulimit -a` A user can view all applied limits with this command

**Sudo privileges**

`sudo visudo` #Modify the /etc/sudoers file - do not edit it directly with "vi"

To give full access, add:
```
username ALL=(ALL:ALL) ALL
```

The first field specifies the username the rule applies to (`%` for groups, e.g., `%developers`).

The first `ALL` means the rule applies to all hosts.

The second indicates that the user can execute commands as all users.

The third indicates that the user can execute commands as all groups.

The last means the rules apply to all commands.

## Group Management

**View groups**

`groups` #To see the groups of our user

`sudo groups john` #To see the groups of another user

**Add a group**

`sudo groupadd group01`

**Delete a group**

`sudo groupdel group01`

**Add/Remove a user from a secondary group**

`sudo gpasswd -a user01 group01` #Add the user to the group

`sudo usermod -a -G group01 user01` #Add a secondary group via usermod

`sudo gpasswd -d user01 group01` #Remove the user from the group

**Change the primary group**

By default, a user's primary group corresponds to a group with the same name as the user.

`sudo usermod -g developers john`

**Rename a group**

`sudo groupmod --new-name programmers developers`

## Permissions
`ls -l` allows viewing permissions. There are 9 characters: the first 3 for the owner, then the group, then other users.

**chmod**

`chmod u+w my_file.jpg` #Add write permission for the owner

`chmod g+x my_file.jpg` #Add execute permission for the group

`chmod o+x my_file.jpg` #Add execute permission for others

`chmod u-w my_file.jpg` #Remove write permission for the owner

`chmod u=rx my_file.jpg` #Set exactly read/execute permissions for the owner

`chmod u=rw,g=r,o= my_file.jpg` #Set RW for the owner, R for the group, and no permissions for others in one command

**chmod in octal**

Permissions can be checked with `stat`.

Example: "Access: (0640/-rw-r-----)"

User=6, group=4, others=0

It's in binary, "rw-" => 110, which equals 6.

`chmod 777 my_file.txt` #Full permissions (1+2+4) for everyone

## Special permissions

**SUID (Set User ID)**

SUID allows executing a file with the owner's permissions, regardless of the user executing it.

It only applies to files, not directories.

When checking user permissions with `ls -l`, the `x` is replaced by `s`.

Two ways to set it:

`chmod u+s my_file.jpg`

`chmod 4755 my_file.jpg` #Octal value is 4

**GUID (Set Group ID)**

GUID also applies to directories, not just files like SUID.

Allows execution with group permissions.

`chmod g+s directory/`

`chmod 2755 directory/` #Octal value is 2

**Sticky Bit**

Prevents directory deletion by anyone other than the owner.

Useful for protecting files in a directory.

`chmod 1777 /tmp`

`chmod o+t /tmp` #Octal value is 1

## Locate, search, manipulate files

### Locate files

**Find files (find)**

`find / -type f -name 'httpd.conf'` #Search for `httpd.conf` file across the system

**By permissions**

`find . -perm 755` #Files with EXACTLY 755 permissions

`find . -perm -600` #Files with AT LEAST 600 permissions

`find . -perm /664` #Files with ANY of these permissions

`find . -perm /u=s` #Files with SUID (regardless of other permissions)

`find . \! -perm -o=r` #Files without read permissions for others (`-not` = `\!`)

`find /opt/assets/ -type f -perm 666 -delete` #Delete files with 666 permissions

**By names/extensions**

`find /my/directory/ -name '*.jpg'` #Only .jpg extensions

`find /my/directory/ -not -name 'f*'` #Files NOT starting with "f"

`find /my/directory/ -iname file.txt` #Using -iname ignores case sensitivity

`find /my/directory/ -type d -name test` #Directories named "test"

`find /opt/findme/ -type f -size +1k -exec cp {} /opt/ \;` #Copy files over 1KB to "/opt/"

**By size**

`find /my/directory/ -size +10M` #Files larger than 10MB

`find /my/directory/ -size 10M` #Files exactly 10MB

`find /usr/ -size +5M -size -10M` #Files between 5MB and 10MB

**By modification date**

`find /dev/ -mmin -5` #Files modified in the last 5 minutes

`find /dev/ -mmin +5` #Files modified before the last 5 minutes

`find /dev/ -mtime 1` #Files modified in the last 24 hours

### Display file contents

**grep**

`grep enabled system.cfg` #Search for "enabled" in the configuration file

**cat and tac**

`cat mytext.txt` #Display content

`tac mytext.txt` #Display content in reverse order

**tail**

`tail -n 20 /var/log/dnf.log` #Show the last 20 lines (default is 10 without the option)

**head**

`head -n 20 /var/log/dnf.log` #Show the first 20 lines (default is 10 without the option)

### Manipulate file content

**sed**
`sed 's/exampel/example/'` #Replace the first occurrence per line of "exampel" with "example"

`sed 's/exampel/example/g'` #Replace all occurrences of "exampel" with "example"

The `-i` (`--in-place`) option modifies the file directly instead of creating a new one.

Without `-i`, it outputs the modified version without altering the original file.

`sed -i '10,50 s/enable/disable/g' filename` #Replace a word only between lines 10 and 50

`sed -i '1,1000d' /home/bob/testfile` #Delete the first 1000 lines

**cut**

file.csv
```
ravi,seattle,usa
mark,toronto,canada
john,newyork,usa
```

`cut -d ',' -f 2 file.csv`

```
seattle
toronto
newyork
```

`-d` specifies the delimiter (comma), `-f` specifies the field (second column, city)

**sort + uniq**

`sort cities.txt | uniq` #sort arranges lines alphabetically, uniq removes consecutive duplicate lines

**diff**

Compare two files:

`diff file1 file2`

`diff -y file1 file2` #Side-by-side comparison


### Searching within Content

**grep**

`grep -i 'Centos' /etc/os-release` #To search for the word centos in the file, -i for case insensitive

`grep '^core' /etc/services` #To search only for words that start with 'core'

`grep -r 'CentOS' /etc/` #To search within an entire directory

`grep -ir 'CentOS' /etc/` #Combine both

`grep -v 'centos' /etc/os-release` #To find lines that do not contain the word (--invert-match)

`grep -w 'red' /etc/os-release` #To search only for the word "red" and not for longer words containing red (e.g.: redhat)

`grep -o 'centos' /etc/os-release` #To display only the searched word and not the entire line (--only-matching)

`grep '\.' file.txt` #To search for a dot, use \ as an escape character

`grep 'centos' /etc/os-release -c` #Counts the number of occurrences of 'centos' (-c is equivalent to wc -l)

**egrep**

`egrep -r '0{3,}' /etc/` #Lines containing at least three "0"s

`egrep -r '10{,3}' /etc/` #Lines containing one "1" and at most three "0"s (including those with no "0"s)

`egrep -r '0{3,5}'` #To give a range: between 3 and 5 "0"s

`egrep -r 'disabled?' /etc/` #To search for both "disable" and "disabled" in the "/etc/" directory

`egrep -r 'enabled|disabled' /etc/` #To search for "enabled" and "disabled"

`egrep -r 'c[au]t' /etc` #Which contains "cat" or "cut"

### Pagers

**less**

*less allows you to have the output in a pager, enabling you to navigate through the text, search for a word, etc. It's an enhanced version of "more".*

`history | less` #Allows you to have the output of the history command in the pager

`less file.txt`

## Input/Output Redirections

**Redirect INPUT**

`./backup.sh < backup_list.txt` #The contents of backup_list.txt are redirected into backup.sh, providing the list of files to be backed up.

**Redirect STDOUT**

`echo "ok" > test.txt` #Redirect stdout

`echo "ok" 1> test.txt` #Same

**Redirect STDERR**

`echoooo 2> /dev/null` #Redirect the error (if we direct it to /dev/null, it's to suppress it and not display it)

**Redirect both in the same file**

`grep -r '^The' /etc/ 1>all_ouput 2>&1`

**Redirect both in different files**

`grep -r '^The' /etc/ 1>output.txt 2>errors.txt`

## Archives : Pack/Unpack

Archive extension = .tar

**Create an archive**

`tar -cf archivename.tar /files-you-want-to-archive`

**Unpack an archive**

`tar -xvf /archivename.tar`

**Add a file to the archive**

`tar -rvf /root/homes.tar /tmp/newfile1`

**Update an archive**

`tar -uvf /root/homes.tar /home` #Write newer versions of all files in /home to the archive

**Check the archive content without unpacking**

`tar -tvf /root/homes.tar`

## Compression and Extraction

Compressed archive extension = .tar.gz

**Compress a file**

`gzip file1` #If you want to keep the original file, you need to use the -k option.

`bzip2 file2`

`xz file file3`

`tar -czf archive.tar.gz file1` #Equivalent = tar --create --gzip --file archive.tar.gz file1

**Decompress a file**

`gzip --decompress file1.gz`

`bzip2 --decompress file2.bz2`

`xz --decompress file3.xz`

`tar -xzf /chemin/vers/archive.tar.gz -C /chemin/vers/dossier_cible` #Decompress the archive with tar and specify the destination location.

## Process Management

*A process is a running instance of a program, each with its own PID and resources.*

**systemctl**

`systemctl list-units --type=service` #List services you can add --all

`sudo systemctl status apache2` #Check the current status of the process

`sudo systemctl stop apache2` #Stop the process

`sudo systemctl start apache2` #Start the process

`sudo systemctl restart apache2` #Restart the process

`sudo systemctl reload apache2` #Reload the configuration files of the process

`sudo systemctl enable apache2` #Enable apache2 from starting on boot

`sudo systemctl disable apache2` #Disable apache2 from starting on boot

`systemctl cat sshd.service` #Check service config

`sudo systemctl edit --full networking.service` #Edit a service

`sudo systemctl revert networking.service` #Reverting back to the vendor default

**Listing Processes (ps, pgrep)**

`ps` #Process running in the current shell

`ps aux` #List all processes in the system

`ps u 1` #List process with pid=1 (u = user oriented format)

`ps u -U test` #List process from user “test”

`pgrep -a syslog` #Search for a process by its name (ps + grep combined)

**lsof (list open files)**

`lsof -p <pid>` #View the files used by a process

`sudo lsof /var/log/messages` #Check which process is using the file

**kill signals**

`kill -L` #List all signals

`kill [pid]`  #The ps command provides the PID. By default, the signal is SIGTERM (terminates the process "gracefully")

`kill -9 [pid]` #To force kill (SIGKILL)

`killall firefox` #Kill all pids with the process name

**manage priority (nice, renice)**

*nice is used to invoke a utility or shell script with a particular CPU priority, thus giving the process more or less CPU time than other processes. A niceness of -20 is the lowest niceness, or highest priority. The default niceness for processes is inherited from its parent process and is usually 0.*

`nice -19 yes > /dev/null &` #Here, the 'yes' command will not be prioritized and will yield resources to other processes

`renice 19 -p [pid]` #To change priority of a running process

## Jobs

*A job is a unit of work managed by a shell, which can be a command or a series of commands executed in the foreground or background.*

`jobs` #See current jobs

**Bring to foreground and background (fg, bg)**

`fg <job_id>` #Resume a suspended process in the foreground

`bg <job_id>` #Resume in the background

`yes > /dev/null &` #Run a process in the background using '&'

**Schedule a one-time job (at)**

`atq` #Display a list of pending tasks

`at '15:30 August 20 2024'` #Schedule a task: enter the command and exit with Ctrl+D

`sudo at '15:30 08/20/2024'` #As root instead of the user

`atrm <jobid>` #Remove a pending task

**Schedule a recurring job (cron)**

Scripts must be executable and should not have an extension (e.g., .sh) to work with cron.

`cat /etc/crontab` #To see the cron syntax

possible operators:

```
* = match all possible values (i.e., every hour)
, = match multiple values (i.e., 15,45)
- = range of values (i.e., 2-4)
/ = specifies steps (i.e., */4)
```

e.g.: `30 21 * * * /usr/bin/touch test_passed` #Every day at 21:30

**edit the cron configuration file. Allows adding, deleting, modifying tasks. -e**

`crontab -e` #Edit the user's crontab

`sudo vi /etc/crontab` #Edit the system-wide crontab

`sudo crontab -l` #View root's crontab without editing it

`sudo crontab -e` #Edit root's crontab, not the logged-in user's

`sudo crontab -e -u username` #Edit another user's crontab (possible as root)

`/etc/cron.hourly` #All scripts in this folder will be executed every hour

**anacron**

Anacron is used to run commands periodically (with a period defined in days).

It is useful for machines that do not run 24/7, such as laptops or workstations.

`/etc/anacrontab` #Edit to add a job

e.g.: `10 5 db_cleanup /usr/bin/touch /root/anacron_created_this` #Run once every 10 days, 5 minutes delay, job ID is "db_cleanup"

`sudo anacron -n -f` #Force execution of all jobs scheduled for the day

**Check job completion**

`sudo cat /var/log/cron` #Job logs (same for cron/anacron)

## Logs

**View systemd logs**

`journalctl`

`journalctl -e` #To go directly to the end

`journalctl /usr/bin/sudo` #See only logs generated by the sudo command (use `which sudo` to get the path)

`journalctl -u ssh.service` #If you know the name of the service, you can see its logs 

`journalctl -f` #Follow mode, to see new logs

`journalctl -p err` #View messages with "error" priority level

`journalctl -p info` #View messages with "info" priority level

`journalctl --since "10 minutes ago"` #To display since a targeted period

`sudo journalctl -g '^c'` #To filter with grep

**Folder /var/log => contains several log files**

`cat /var/log/auth.log` #e.g.: check authentication logs

`cat /var/log/syslog | grep eth0` #e.g.: Check what concerns eth0 in syslog

**View kernel-related messages**

`dmesg -H` #-H makes the information more readable
 
## System Info 

**Get information about the OS/Hardware**

`uname -a` #Provides general information about the system kernel

`cat /etc/os-release` #Provides information specific to the Linux distribution

**hostnamectl (for systemd OS)**

`hostnamectl` #hostname, transient hostname, etc.

`sudo hostnamectl set-hostname my-new-hostname` #Change the hostname

**uptime**

`uptime` #View uptime + load average => (last minute, last 5min, last 15min)

**lscpu, lspci**

`lscpu` #CPU info

`lspci` #PCI device info

## Task manager 

*View process consumption, etc.* 

`top` 

`htop` #Interactive process viewer (not installed by default)

**RAM Memory**

`free -h` #View available RAM (-h to get results in GB)

`free --mega`

**Folder /proc/ => contains files with information about hardware**

`cat /proc/meminfo` #Contains information about RAM

## File Systems

### Checking usage

**Check available disk space**

`df -h` #View free space (-h = human-readable format)

`df -ht ext4` #Only shows ext4 partitions, displayed in GB

`df -T /mnt` #See information about the fs that contains the folder (including the file system)

**Evaluate disk space occupied by a file**

`du -sh /bin/`

### Repair FS

`xfs_repair -v /dev/vdb1` #For xfs

`sudo fsck.ext4 -v -f -p /dev/vdb2` #For ext4

`sudo e2fsck -f /dev/vdb2` #Provides more specialized checks

### Partitions

`lsblk` #List block devices. (-f useful to see filesystem type)

`sudo fdisk --list /dev/sda` #List the partitions of disk sda

`sudo cfdisk /dev/sda` #Modify partitions (size, type)

### Swap Space

*A swap partition is a feature in Linux that provides virtual memory space and multiple benefits. The EFI system partition (also called ESP) is an OS independent partition that acts as the storage place for the UEFI boot loaders*

`swapon --show` #To verify the swap device (swap partition or file)

`sudo mkswap /dev/vdb3` #Format partition as swap

`sudo swapon /dev/vdb3` #Use the partition as swap memory (changes lost on reboot)

`sudo swapoff /dev/vdb3` #Stop using it

`sudo dd if=/dev/zero of=/swap bs=1M count=128` #Create a swap file

Increase swap file size by 1GB:

```sh
sudo dd if=/dev/zero of=/swapfile bs=1M count=1024 oflag=append conv=notrunc
sudo swapoff /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

`/dev/vdb none swap defaults 0 0` #To put in /etc/fstab to use the swap at boot

### Create FS

**ext**

`man mkfs.ext4` #Documentation for ext4

`sudo mkfs.ext4 /dev/sdb1` #Create an ext4 type filesystem

`sudo mkfs.ext4 -L "BackupVolume" -N 5000 /dev/sdb2` #With a Label + custom Inodes number

`sudo tune2fs -l /dev/sdb2` #View filesystem attributes

`sudo tune2fs -L "SecondFS" /dev/sdb2` #Set a label 

**xfs**

`man mkfs.xfs` #Documentation for xfs

`sudo mkfs.xfs /dev/sdb1` #Create an xfs type filesystem

`sudo xfs_admin -L "newLabel" /dev/sdb1` #Change label

### Mount FS

`findmnt` #Shows everything mounted on the system

`findmnt /dev/vda1` 

`findmnt -t xfs,ext4`

**Non-Permanent**

`sudo mount /dev/vdb1 /mnt/` #Mount FS

`sudo mount -o ro /dev/vdb2 /mnt` #Mount FS as Read-Only

`sudo mount -o remount,rw /dev/vdb2 /mnt` #Change option

`sudo umount /mnt` #Unmount FS

**Permanent**

`sudo blkid /dev/sdb1` #To know the UUID of a partition

`sudo vim /etc/fstab` #To mount a FS at system boot. Fill according to this format:

```
<device> <mount point> <filesystem type> <options> <dump> <pass>

<device>: The device or partition to be mounted. This can be the device name (like /dev/sda1), UUID, or label.
<mount point>: The directory where the filesystem will be mounted.
<filesystem type>: The type of the filesystem (e.g., ext4, ntfs, nfs).
<options>: Mount options, such as rw for read-write, defaults, noauto, etc.
<dump>: Used by the dump command to decide whether to back up the filesystem. Usually set to 0.
<pass>: Used by the fsck command to determine the order of filesystem checks at boot time. Typically set to 1 for root filesystems and 2 for others.
```

Example:
`UUID=your-uuid /backups xfs defaults 0 0`

`sudo mount -a` #To apply changes (otherwise, on the next reboot)

**On Demand**

`sudo dnf install autofs`  + `sudo systemctl enable autofs.service --now`

`sudo dnf install nfs-utils` + `sudo systemctl start nfs-server.service --now`

Controls which file systems are exported to remote hosts:

`sudo vim /etc/exports` => `/etc 127.0.0.1(ro)`

`sudo systemctl reload nfs-server`

Autofs consults the master map configuration file `/etc/auto.master` to determine which mount points are defined:

`sudo vim /etc/auto.master` => `/shares /etc/auto.shares --timeout=400`

`sudo vim /etc/auto.shares` => `mynetworkshare -fstype=auto 127.0.0.1:/etc`

`sudo systemctl reload autofs`

### Remote FS

**Server Side**

`sudo apt install nfs-kernel-server`

`sudo vim /etc/exports` #Choose the locations to share

```
<path> <hostname(rw,sync,no_subtree_check)> <hostname2(rw,sync,no_subtree_check)
```

`sudo exportfs -r` #Re-export to apply changes

`sudo exportfs -v` #To see active shares

**Client Side**

`sudo apt install nfs-common`

`sudo mount <IP/Hostname of server>:/path/to/remote/directory path/to/local/directory`

`sudo mount server1:/etc /mnt`

For auto mount:

`sudo vim /etc/fstab` => `127.0.0.1:/etc/mnt nfs defaults 0 0`

### Advanced Permissions

#### ACL

`sudo setfacl --modify user:username:rw examplefile` #Modifies the access control list (ACL) of the file examplefile, granting read and write permissions (rw) to the user 'username'.

`sudo setfacl --remove user:aaraon examplefile` #Remove their rights

`getfacl examplefile` #Check ACL

#### chattr 

**Change Attribute**

`sudo chattr` #The +a option sets the "append-only" attribute on a file, meaning it can only be opened in "append" mode for writing once set.

`sudo chattr +i newfile`  #When marked as immutable, a file cannot be modified, deleted, renamed, or linked to by any user, including root. Can be removed with 'chattr -i' first.

`lsattr newfile` #Check attributes

### Disk Quotas

`sudo dnf install quota`

`sudo vim /etc/fstab`

```
/dev/vdb1 /mybackups xfs defaults,usrquota,grpquota 0 2
```

The quotas will apply on the next boot.

`sudo quotacheck --create-files --user --group /dev/vdb2` #Performs a filesystem quota consistency check on the specified device (/dev/vdb2) and creates the necessary files (aquota.user and aquota.group)

`sudo quotaon /mnt/` #Enables disk quotas on the filesystem mounted at /mnt/

`sudo edquota -u username` #This opens a text editor where you can set the soft and hard limits for the user's disk usage

`sudo edquota --group grpname` #For groups

`sudo quota --user username` #Check quota for a specific user

`sudo quota --group grpname` #Check quota for a specific group

`fallocate --length 100M /mybackups/username/100Mfile` #Create a 100M file. Useful for testing the quota

`sudo quota --edit-period`

## Monitor Storage Performance

`sudo apt install sysstat`

`iostat -h` #View metrics since server boot

`iostat -h 10` #View stats for the last 10 seconds (auto-refresh)

`pidstat -d` #To see which process is causing I/O

`pidstat -d 1` #To see the last second (auto-refresh) - note the process number

`ps <pid>` #See the command performing the I/O

## LVM Storage

`sudo dnf install lvm2 -y`

`man lvm` #All documentation on LVM

```
PV : Physical Volumes Hard disks, hard disk partitions, RAID volumes, or logical units from a SAN form "physical volumes" (PV).

VG : Volume Groups These physical volumes (one or more) are concatenated into "volume groups" (VG). These VG are equivalent to pseudo-hard drives.

LV : Logical Volumes "Logical volumes" (LV) are then sliced from the volume groups, formatted, and mounted in filesystems. The LV are equivalent to pseudo-partitions.

FS : File Systems "File systems" (FS) are a way to store information and organize it in files. They have a mount point and a type (ext4, xfs, btrfs).
```

### Physical Volumes (PV)

`sudo pvs` #View the PVs

`sudo pvdisplay` #View more information

`sudo lvmdiskscan` #Identify available disks/partitions

`sudo pvcreate /dev/sdc /dev/sdd` #Create a PV

`sudo pvremove /dev/sdd` #Remove PV

### Virtual Group (VG)

`sudo vgs` #View the VGs

`sudo vgdisplay` #View more information

`sudo vgcreate my_volume /dev/sdc /dev/sdd` #Create a VG

`sudo vgextend my_volume /dev/sde` #Extend VG

`sudo vgreduce my_volume /dev/sde` #Reduce VG

### Logical Volumes (LV)

Similar to a partition, but for an LVM.

`sudo lvs` #View the LVs

`sudo lvdisplay` #View the LVs

`sudo lvcreate --size 2G --name partition1 my_volume` #Create LV

`sudo lvresize --extents 100%VG my_volume/partition1` #Extend to all available space

`sudo lvresize --size 2G my_volume/partition1` #Resize to 2G

`sudo mkfs.xfs /dev/my_volume/partition1` #Finally, create a FS on it

`sudo lvresize --resizefs --size 3G my_volume/partition1` #Extend LV to 3G as well as the FS. Without `--resizefs`, only the LV is extended.

`sudo lvremove my_volume/partition1` #Remove LV

## Network Block Device

**Server Side**

`sudo apt install nbd-server`

`sudo vim /etc/nbd-server/config`

```
[partition1]
  exportname = /dev/vda1
```

`sudo systemctl restart nbd-server.service`

`man 5 nbd-server` #See documentation on configuration files

**Client Side**

`sudo apt install nbd-client`

`sudo modprobe nbd` #To be redone at each boot

`sudo vim /etc/modules-load.d/modules.conf` => add `nbd` #to avoid repeating the command every time

`sudo nbd-client 161.35.112.18 -N partition1`

`sudo mount /dev/nbd0 /mnt`

`sudo nbd-client -d /dev/nbd0` #Detach block device

## Encrypted Storage

`sudo cryptsetup luksFormat /dev/vde` #Format encrypted storage

`sudo cryptsetup luksChangeKey /dev/vde` #Change the key

`sudo cryptsetup open /dev/vde` #Open the encrypted storage

`sudo cryptsetup close mysecuredisk` #Close the encrypted storage

## RAID Devices

`sudo mdadm --create /dev/md0 --level=0 --raid-devices=3 /dev/vdc /dev/vdd /dev/vde` #Create a RAID 0 array using 3 devices

`sudo mkfs.ext4 /dev/md0` #Create an FS on it

`sudo mdadm --stop /dev/md0` #Stop the RAID array

`sudo mdadm --zero-superblock /dev/sdc /dev/vdd /dev/vde` #Repurposing disks that were previously part of RAID arrays.

`sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdc /dev/vdd`

`sudo mdadm --manage /dev/md0 --add /dev/vde` #Add a 3rd disk

`cat /proc/mdstat` #Current RAID configuration and status

## Boot

### Reboot / Poweroff
`sudo systemctl poweroff` #Power off the system

`sudo systemctl reboot` #Reboot the system

`sudo systemctl poweroff --force` #Force power off (can use --force twice to power off as if pressing the button)

`sudo shutdown 02:00` #Shut down at 2 AM

`sudo shutdown +15` #Shut down in 15 minutes

`sudo shutdown -r 02:00` #Reboot at 2 AM

`sudo shutdown -c` #Cancel a scheduled shutdown

### Change Boot Behavior

`sudo systemctl get-default` #View the current mode

`sudo systemctl set-default multi-user.target` #Change to a mode without a graphical interface

`sudo systemctl isolate graphical.target` #Switch back to graphical mode (without rebooting)

### Grub Installation

At boot, choose "troubleshooting" then "rescue".

`chroot /mnt/sysroot` #Change root to the system's root

`grub2-mkconfig -o /boot/grub2/grub.cfg` #Generate Grub config

`grub2 install /dev/sda` #Install Grub to the specified device

Reboot into the OS

`sudo vim /etc/default/grub` #Edit Grub settings

`sudo grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg` #Regenerate Grub config file

## Create a systemD Service
`/lib/systemd/system/` #Directory for systemd services

**Example with a Bash Script**

```sh
#!/bin/sh
echo "MyApp Started" | systemd-cat -t MyApp -p info
echo "MyApp Crashed" | systemd-cat -t MyApp -p err
```

`sudo chmod a+x /usr/local/bin/myapp.sh` #Make the script executable

**Create the Service**

Let’s create a file called `/etc/systemd/system/MyApp.service`

```
[Unit]
Description=My Application
After=network.target

[Service]
ExecStartPre=echo "SystemD  is preparing MyApp"
ExecStart=/usr/local/bin/myapp.sh
KillMode=process
Restart=always
RestartSec=1
Type=simple


[Install]
WantedBy=multi-user.target
```

`sudo systemctl daemon-reload` #Reload systemd to recognize the new service

## Kernel Runtime

`sudo sysctl -a` #Display all available values

`sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1` #Non-persistent change

To make a change persistent:

`sudo vim /etc/sysctl.conf` #You can add a line in the main config file, e.g., "vm.swappiness=5"

`sudo vim /etc/sysctl.d/<name>.conf` #You can also create a new .conf file in this folder and put the parameter inside. Example:

```
net.ipv6.conf.all.disable_ipv6 = 1 
net.ipv6.conf.default.disable_ipv6 = 1
```

This allows you to create individual configuration files for better organization

## SELinux MAC 

**Mandatory Access Control**

**Check the Current Mode**

`getenforce` #See if SELinux is enabled

`sestatus` #Alternative command

**Change the Mode**

`sudo setenforce 1` #With setenforce: 1=Enforcing / 2=Permissive

`sudo vi /etc/selinux/config` #Directly edit the config file

**Disable AppArmor and Install SELinux for Non-RH Distros**

If using other distros (Ubuntu, Debian, etc.), it's better to disable AppArmor if you want to install SELinux.

`systemctl status apparmor` #Check if AppArmor is enabled (it usually is by default)

`sudo systemctl stop apparmor.service` #Stop the service

`sudo systemctl disable apparmor.service` #Disable it on boot

`sudo apt install selinux-basics auditd` #Install SELinux

`sudo selinux-activate` #Prepares the bootloader (GRUB). On the next reboot, all files will be updated (autorelabel).

**Display Security Contexts**

`seinfo -r` #To see all roles. Use -u for users, -t for types

`ls -Z` #For files

user : role : type : level

`ps axZ` #For processes

`id -Z` #For the user

**Display Mappings**

`sudo semanage login -l` #List the SELinux login mappings on a system

`sudo semanage user -l` #List the SELinux user mappings on a system

**Change Security Contexts (user, role, type)**

`sudo chcon -t httpd_sys_content_t /var/index.html` #Change the SELinux context of "/var/index.html" file to type "httpd_sys_content_t"

`sudo chcon -u unconfined_u /var/log/auth.log` #Change user

`sudo chcon -r object_r /var/log/auth.log` #Change role

`sudo chcon --reference=/var/log/syslog /var/log/auth.log` #To set the same parameters as an existing file

`sudo restorecon -F -R /var/www/` #To restore the default security contexts on the folder (-R for recursive)

`sudo semanage fcontext --add --type var_log_t /var/www/10` #Change the default context but without applying it. If restored with restorecon, it will take these.

**View SELinux Logs**

`sudo audit2why --all` #View SELinux logs and explanations

**Allow Module**

`sudo audit2allow --all -M mymodule` #Create a module to allow previously denied operations

## Essential Networking Commands

**Network Interfaces**

`ip addr` #Information on different interfaces: @IP @MAC, etc.

`sudo iftop -i eth0` #View real-time bandwidth on the 'eth0' interface (install the 'iftop' package beforehand)

**Socket Statistics (ss)**

Replaces `netstat`: visualize open network ports, active connections, listening services, etc.

`sudo ss -tunlp` #-l for listening, -t for TCP connections, -u for UDP connections, -n for numeric values

`sudo ss -tunlp | grep LISTEN` #To see open ports

**Netcat (nc)**

Replaces `telnet`: Netcat or NC is a utility tool that uses TCP and UDP connections to read and write in a network.

It is also very useful for connectivity testing!

`nc -help` #Command help

`nc -z -v mywebsite.com 80` #Test port 80

**Traceroute**

Follow the path a packet takes through a network to reach a given destination

`traceroute www.example.com` #Basic traceroute

`traceroute -p 80 www.example.com` #This command uses TCP packets on port 80 instead of UDP packets

**Nmap**

Network scan, scan ports

`nmap -v -sT localhost` #Scan your PC to see open ports

`nmap -sn 10.172.16.0/24` #Subnet scan to discover hosts without scanning ports (fast)

**DNS Queries**

*Both dig and nslookup are command-line tools used for querying DNS (Domain Name System) servers to retrieve information about domain names.*

**Dig**

`dig example.com` #Basic DNS Lookup

`dig +trace example.com` #Detailed DNS Lookup

`dig example.com @8.8.8.8` #Querying a Specific DNS Server

`dig -x 8.8.8.8` #Reverse DNS Lookup

**Nslookup**

`nslookup example.com` #Basic DNS Lookup

`nslookup example.com 8.8.8.8` #Querying a Specific DNS Server

## Configure Network Interfaces

### Non-Persistent (ip addr)

Changes are not saved upon reboot

`sudo ip link set dev enp0s8 up` #Activate an interface (dev for "device")

`sudo ip addr add 192.168.5.55/24 dev enp0s8` #Configure the interface with IP/Mask CIDR

`sudo ip addr delete 192.168.5.55/24 dev enp0s8` #Remove the IP

### Persistent with netplan (Ubuntu)

*Netplan is typically used on Ubuntu and its derivatives since Ubuntu 17.10 (Artful Aardvark). It is used as a high-level configuration abstraction for various backends, including NetworkManager and systemd-networkd.*

`ls /usr/share/doc/netplan/examples` #To see examples and understand how to configure network interfaces. Check static.yml or dhcp.yml

`netplan get` #View the network interface configuration

`sudo vim /etc/netplan/01-netcfg.yaml` #Configure a static IP

```yaml
network:

    Version: 2
    Renderer: networkd
    ethernets:
       DEVICE_NAME:
          Dhcp4: yes/no
          Addresses: [IP/NETMASK]
          Gateway: GATEWAY
          Nameservers:
             Addresses: [NAMESERVER, NAMESERVER]
```

`sudo netplan try` #Test the configuration

`sudo netplan apply` #Apply the new configuration directly without testing



### Persistent with nmcli (CentOS)

*nmcli is typically used on distributions that use NetworkManager for network configuration. This includes CentOS, Fedora, Ubuntu (although Ubuntu primarily uses netplan now, it still supports nmcli), and other distributions.*

`nmcli con add con-name <connection_name> ifname <interface_name> type ethernet` #Creates a new connection with the specified name and interface name for an Ethernet connection.

`nmcli con mod <connection_name> ipv4.method manual ipv4.addresses <ip_address>/<subnet_mask> ipv4.gateway <gateway_ip> ipv4.dns <dns_server>` #Modifies the IPv4 settings of the specified connection to use a manual configuration with the given IP address, subnet mask, gateway IP, and DNS server.

`nmcli con mod <connection_name> ipv4.method auto` #Changes the IPv4 configuration method of the specified connection to automatic.

`nmcli con up <connection_name>` #Brings up the specified connection.

### Persistent with /etc/network/interfaces (Debian)

`sudo vim /etc/network/interfaces`

```
auto eth0:0
iface eth0:0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    network 192.168.1.0
    broadcast 192.168.1.255
    gateway 192.168.1.1
```

`sudo service network restart`

### Bridge Network

**With Netplan (Ubuntu)**

`cat /usr/share/doc/netplan/examples/bridge.yml` #View an example to understand the config

`cp /usr/share/doc/netplan/examples/bridge.yml /etc/netplan/99-bridge.yml` #Use as a template

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s9:
      dhcp4: no
      dhcp6: no
    enp0s10:
      dhcp4: no
      dhcp6: no
  bridges:
    br0:
      dhcp: yes
      interfaces:
        - enp0s9
        - enp0s10
```

### Bond Network

**With Netplan (Ubuntu)**

`cp /usr/share/doc/netplan/examples/bond.yml /etc/netplan/99-bond.yml` #Same as above

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s9:
      dhcp4: no
      dhcp6: no
    enp0s10:
      dhcp4: no
      dhcp6: no
  bonds:
    bond0:
      dhcp: yes
      interfaces:
        - enp0s9
        - enp0s10
      parameters:
        mode: active-backup
        primary: enp0s9
```

`cat /proc/net/bonding/bond0` #To get more info

## Configure Routes

### Non-Persistent (ip route)

`ip route show` #View defined routes

These changes are not persistent*

`sudo ip route add <destination_network>/<netmask> via <gateway_ip>` #Add a route

`sudo ip route del <destination_network>/<netmask> via <gateway_ip>` #Remove a route

`sudo ip route add default via <gateway_ip>` #Set a default route (gateway)

`ip route get <destination_ip>` #View route details for a specific destination

`sudo ip route change <destination_network>/<netmask> via <new_gateway_ip>` #Change a route

`sudo ip route add <destination_network>/<netmask> dev <interface_name>` #Rule based on the interface

### Persistent with Netplan (Ubuntu)

Define routes in the netplan file (YAML format)

`sudo vi /etc/netplan/<configuration_file>.yaml`

```yaml
routes:
        - to: 0.0.0.0/0 # Route par défaut pour toutes les adresses
          via: 192.168.1.1 # Passerelle pour la route par défaut
          on-link: true # la route spécifiée est directement accessible via une interface réseau disponible localement sur le même sous-réseau.
```

`sudo netplan apply` #Apply changes

### Persistent with nmcli (CentOS)

`nmcli device show | grep -i gateway` #View routes (or use 'ip route show')

`nmcli connection show` #View the interface used

`nmcli connection modify <connection_name> +ipv4.routes "<destination_cidr> <gateway_ip>"` #Add a route

`sudo nmcli connection modify eth1 +ipv4.routes "192.168.0.0/24 172.28.128.100"`  #Example ^

`sudo nmcli device reapply eth1` #Apply changes

`nmcli connection modify <connection_name> -ipv4.routes "<destination_cidr> <gateway_ip>"` #Remove a route

`nmcli connection modify <connection_name> +ipv4.gateway <gateway_ip>` #Set a default route (gateway)

`nmcli connection modify <connection_name> -ipv4.gateway` #Remove a default route

Alternative with the graphical interface

`sudo nmtui` #Open NetworkManager TUI

`sudo nmcli device reapply eth1` #Reapply changes

### Persistent with Network Scripts (CentOS)

`sudo nano /etc/sysconfig/network-scripts/route-<interface>` #Edit route file

`<destination_network> via <gateway>` #Line to add

## Firewall 

### UFW (Ubuntu)

*Uncomplicated Firewall*

**Check Status**

`sudo ufw status` #See if UFW (Uncomplicated Firewall) is enabled

`sudo ufw status verbose` #Verbose for detailed version.

`sudo ufw status numbered` #See the numbers of firewall rules

By default, everything is disabled. It's better to create a rule for SSH before enabling it.

Rules are applied according to their numbered order.

**Create/Delete Rules**

`sudo ufw enable` #Enable UFW 

`sudo ufw allow 22` #Allow SSH for everyone

`sudo ufw allow 22/tcp --permanent` #Permanent rule (otherwise, change lost on next reboot)

`sudo ufw allow from 192.168.1.60 to any port 22` #Allow SSH for an IP (any = the receiver = any IP on the server)

`sudo ufw allow from 10.11.12.0/24 to any port 22` #Allow SSH for an IP range

`sudo ufw allow from 10.11.12.0/24` #Allow everything for an IP range

`sudo ufw insert 3 deny 10.11.12.100` #Block an IP address on all ports + Place the rule at number #3

`sudo ufw delete 1` #Delete firewall rule number 1

`sudo ufw delete allow 22` #Delete the rule with IPv4 and IPv6 at the same time

`sudo ufw deny out on enp0s3 to 8.8.8.8` #Block all outgoing traffic from the network card enp0s3 to 8.8.8.8

### Firewalld (CentOS)

**Check zones and firewall rules**

`sudo firewall-cmd --get-default-zone` #View the default zone

`sudo firewall-cmd --set-default-zone=public` #Change the zone

`sudo firewall-cmd --list-all` #View current firewall rules

**Create/Delete Rules**

`sudo firewall-cmd --add-port=7869/tcp` #Allow a port

`sudo firewall-cmd --add-service=https` #Allow a service (equivalent to --add-port=443)

`sudo firewall-cmd --info-service=https` #View ports used by the service

`sudo firewall-cmd --remove-port=53/udp` #Remove a port

`sudo firewall-cmd --remove-service=http` #Remove a service

**Trusted Zone**

`sudo firewall-cmd --add-source=10.11.12.0/24 --zone=trusted --permanent` #Allow a range to the trusted zone permanently

`sudo firewall-cmd --get-active-zones` #View active zones

`sudo firewall-cmd --remove-source=10.11.12.0/24 --zone=trusted` #Remove the range from the trusted zone

**Make Rules Permanent**

Changes made without the `--permanent` flag are not persistent and will be lost after a system reboot.

To make the current configuration permanent: 

`sudo firewall-cmd --runtime-to-permanent`

It is also possible to add a rule directly to the permanent configuration:

`sudo firewall-cmd --add-port=12345/tcp` #First, add the rule for the active session

`sudo firewall-cmd --add-port=12345/tcp --permanent` #Then, make the rule permanent

### nftables

*Nftables is providing filtering and classification of network packets/datagrams/frames. It aims to replace the existing iptables.*

`nft list tables` #Lists all tables in the nftables configuration.

`nft add table inet my_table` #Adds a new table named "my_table" in the "inet" address family.

`nft delete table inet my_table` #Deletes the table named "my_table" from the "inet" address family.

`nft list ruleset` #Lists all rules in the nftables ruleset.

`nft add rule inet my_table my_chain tcp dport 80 accept` #Adds a rule to the chain "my_chain" in the "my_table" table to accept TCP traffic on port 80.

`nft delete rule inet my_table my_chain tcp dport 80 accept` #Deletes the specified rule from the "my_chain" chain in the "my_table" table.

`nft flush table inet my_table` #Flushes all rules from the "my_table" table in the "inet" address family.

`nft add chain inet my_table my_chain { type filter hook input priority 0 \; }` #Adds a new chain named "my_chain" to the "my_table" table in the "inet" address family with filter type and input hook.

`nft delete chain inet my_table my_chain` #Deletes the chain named "my_chain" from the "my_table" table in the "inet" address family.

### iptables

`iptables -L` #Lists all rules in the iptables configuration.
 
`iptables -A INPUT -p tcp --dport 80 -j ACCEPT` #Appends a rule to accept TCP traffic on port 80 to the INPUT chain.
 
`iptables -D INPUT -p tcp --dport 80 -j ACCEPT` #Deletes the specified rule from the INPUT chain that accepts TCP traffic on port 80.
 
`iptables -F` #Flushes all rules from all chains.
 
`iptables -N my_chain` #Creates a new chain named "my_chain".
 
`iptables -X my_chain` #Deletes the chain named "my_chain".

## Port Redirection

**Enable port forwarding**

Edit `/etc/sysctl.conf` or `/etc/sysctl.d/99-sysctl.conf`

`sudo vi /etc/sysctl.d/99-sysctl.conf` #Uncomment the line for forwarding on IPv4 or IPv6

`sudo sysctl --system` #Reload the configuration

`sudo sysctl -a | grep forward` #Check if the changes are applied

## DNS Resolution

`sudo vi /etc/hosts` #Map domain names to IP addresses before DNS resolution is applied

`resolvectl status` #Check the status of the DNS resolution service

### With SystemD

`sudo vi /etc/systemd/resolved.conf` #Configure DNS resolution on systemd

```
DNS=4.4.4.4 8.8.8.8
```

`sudo systemctl restart systemd-resolved.service` #Apply the changes

### Without SystemD

`sudo vi /etc/resolv.conf` #Classic configuration file. No need to restart the service

```
nameserver 8.8.8.8
```


`resolvectl dns` #Check the DNS servers being used



## Caching DNS Server

### BIND

`sudo dnf install bind bind-utils` #Install BIND

`man named.conf` #Documentation

`sudo vi /etc/named.conf` #Configure BIND

```
listen-on port 53 {127.0.0.1; @IP}; #any pour tout autoriser
allow-query {localhost; @IP}; 
recursion yes;
```

`sudo systemctl start named.service && sudo systemctl enable named.service` #Start and enable BIND service

`sudo firewall-cmd --add-service=dns --permanent` #Allow DNS service through firewall

### Maintain a DNS Zone

Required changes so that local zone file changes can take effect:

`sudo vi /etc/named.conf`

```
zone "example.com" IN {
    type hint;
    file "example.com.zone"
};
```

To create new zones:

`sudo cp --preserve=ownership /var/named/named.localhost /var/named/example.com.zone` #Use named.localhost as a template (maintain file ownership)

## NTP

**Configure the Time Zone**

`timedatectl list-timezones` #View the time zones

`sudo timedatectl set-timezone America/New_York` #Choose a time zone

`timedatectl` #Check if the change has been applied

**Synchronize System Time**

with systemd-timesyncd:

`sudo apt install systemd-timesyncd` #To get synchronization

`systemctl status systemd-timesync.service` #Should return "Active: active (running)"

with chrony and timedatectl:

`sudo yum install chrony -y` 

`sudo systemctl start chronyd`

`sudo systemctl enable chronyd`

`sudo timedatectl show`

`sudo timedatectl set-ntp true`

**Change the NTP Used**

`sudo vim /etc/systemd/timesyncd.conf` #Uncomment 'NTP='

```
[Time]
NTP=0.us.pool.ntp.org 1.us.ntp.org
```

`sudo systemctl restart systemd-timesyncd` #For the change to take effect

`timedatectl show-timesync` #Check that it's okay

## Nginx

### reverse-proxy

An intermediary between the client and the server.

`sudo apt install nginx`

`sudo vim /etc/nginx/sites-available/proxy.conf`

```
server{
  listen 80;
  location/{
    proxy_pass http://1.1.1.1;
  }
}
```

to enable:  `sudo ln -s /etc/nginx/sites-available/proxy.conf /etc/nginx/sites-enabled/proxy.conf`

to remove: `sudo rm /etc/nginx/sites-enabled/default`

to test config files for error : `sudo nginx -t`

to apply our settings : `sudo systemctl reload nginx.service`

### load-balancer

`sudo vim /etc/nginx/sites-available/lb.conf`

```
upstream mywebserver{
  server 1.2.3.4;
  server 5.6.7.8;
}
  server{
    listen 80;
    location /{
      proxy_pass http://mywebserver;
    }
  }
```

`sudo ln -s /etc/nginx/sites-available/lb.conf /etc/nginx/sites-enabled/lb.conf`

`sudo nginx -t`

`sudo systemctl reload nginx.service`

## SSH

### Configure the Server

`dnf install openssh-server` #Install package

`systemctl enable --now sshd.service` #Start and enable on boot

`sudo vim /etc/ssh/sshd_config` #For server configuration

```
PermitRootLogin no # it is recommended not to allow SSH as root
Port 22 # you can uncomment and choose another port
```

`sudo systemctl reload sshd.service` # restart the service for changes to take effect

`man sshd_config` #To search for specific information

### Configure the Client

`sudo vim /etc/ssh/sshd_config`  #For the client (scope = all users)

`sudo vim /home/username/.ssh/config` #Scope = username

`sudo chmod 600 .ssh/config` #For security reasons

`sudo vim /etc/ssh/ssh_config.d/99-our-settings.conf` #To create your settings (e.g., port 229)

### Generate SSH Client Key

`ssh-keygen` #Generate SSH key pair

Private key : `.ssh/id_rsa`

Public key : `.ssh/id_rsa.pub`

`ssh-copy-id username@10.11.12.9` #Copy public key to the server in .ssh/authorized_keys

## PAM 

**Pluggable Authentication Modules**

/etc/pam.d/ # directory for PAM files

`vim /etc/pam.d/su`
```
auth    sufficient    pam_wheel.so    trust   use_uid
```

`man pam_listfile` #See the documentation for a specific module

## LDAP

**Lightweight Directory Access Protocol**

Use an existing LDAP

`sudo apt update && sudo apt  instal libnss-ldapd -y`

Set the LDAP URI, for example:

`ldap://10.21.38.218/`

`getent passwd --service ldap` #View LDAP users only

`getent group --service ldap`

`sudo pam-auth-update` #Change parameters such as creating a home directory on login

## Proxy

**Squid setup**

`sudo dnf install squid` 

`sudo systemctl enable --now squid`

**Configuration**

`sudo vim /etc/squid/squid.conf`

## OpenSSL

**Generate Request + Private Key**

`man openssl-req`

`openssl req -newkey rsa:2048 -keyout key.pem -out req.pem`

**Generate the .crt (self-signed)**

`openssl req -x509 -noenc -newkey rsa:4096 -days 365 -keyout myprivate.key -out mycertificate.crt`

**View the certificate**

`openssl x509 -in mycertificate.crt -text`

## Database

**MariaDB**

MariaDB is a fork of MySQL

`sudo dnf install mariadb-server` #Install

`sudo systemctl enable mariadb-server --now` #Enable service

`sudo firewall-cmd --add-service=mysql --permanent` #If another PC/Server needs access

**Configuration**

`mysql_secure_installation` #Allows you to disable remote root logins, etc.

`cat /etc/my.cnf`

`sudo vim /etc/my.cnf/mariadb-servver.cnf` #Main config file

## Apache

**HTTP Server: Installation**

`sudo dns install httpd`

`sudo systemctl enable httpd --now`

```sh
sudo firewall-cmd --add-service=http
sudo firewall-cmd --add-service=https
sudo firewall-cmd --runtime-to-permanent
```

**Configuration**

`man httpd.conf`

`ls /etc/httpd/conf.d/`

`cat /etc/httpd/conf/httpd.conf` #Main config file

`sudo vi /etc/httpd/conf.modules.d/XX-module.conf` #Module configuration

**Configure 2 sites:**

`sudo vim /etc/httpd/conf.d/two-website.conf`

```
<VirtualHost *:80>
  ServerName store.example.com
  DocumentRoot /var/www/store/
</VirtualHost>

<VirtualHost *:80>
  ServerName blog example.com
  DocumentRoot /var/www/blog/
</VirtualHost>
```

`apachectl configtest` #Check the configuration in place

`sudo systemctl reload httpd.service`

**SSL**

`sudo vim /etc/httpd/conf.d/ssl.conf`

```
<VirtualHost *:443>
  ServerName www.example.com
  SSL Engine on
  SSLCertificateFile "/path/to/file.cert"
  SSLCertificateKeyFile "/path/to/file.key"
</VirtualHost>
```

**Logs**

`sudo vim /etc/httpd/conf/httpd.conf` 

ErrorLog /path/ => To change the log path

LogLevel warn => To choose the log level

**Logs for Different Websites**

```
<VirtualHost *:80>
  ServerName store.example.com
  DocumentRoot /var/www/store/
  CustomLog /var/log/httpd/store.example.com_access.log combined
  ErrorLog /var/log/httpd/store.example.com_error.log
</VirtualHost>
```

## Email

**Mail Server**

`sudo dns install postfix` + `sudo systemctl enable postfix --now` #Install and enable service

`sendmail test@locahost <<< "Testing email"` #Test mail

`cat /var/spool/mail/test`

**Alias**

`sudo vim /etc/aliases` #Add: "alias_name: username" at the end

Example:

```
advertising: test

john: john@example.com
```

`sudo newaliases` #To refresh new aliases

`sendmail advertising@locahost <<< "Testing email ALIAS"` #Test mail that will arrive in the mailbox of 'test'


## IMAP 

**Internet Message Access Protocol**

**IMAP Server installation**
`sudo dns install dovecot` + `sudo systemctl enable dovecot --now` #Install and enable service

If you need to allow through the firewall:
```sh
sudo firewall-cmd --add-service=imap
sudo firewall-cmd --add-service=imaps
sudo firewall-cmd --runtime-to-permanent
```

**Configuration**

`sudo vim /etc/dovecot/dovecot.conf` #Main config file

uncomment: `#protocols = imap pop3 lmtp submission` + `listen = <IP>`

`ls /etc/dovecot/conf.d/` #Where configuration files for Dovecot are stored

## Docker

`docker --help` #Command help

### images

`docker image pull debian` #Pull an image

`docker image pull nginx:1.22.1` #Pull an image with a tag (no tag = latest by default)

`docker rmi nginx:1.22.1` #Remove an image (-f to force image removal)

Example of a Dockerfile to use for creating an image:

```dockerfile
FROM nginx:stable-alpine3.17-slim
COPY index.html /usr/share/nginx/html

EXPOSE 80 
CMD ["nginx", "-g", "daemon off;"]
```

`docker build .` #Build image ( . = if in the directory where the Dockerfile is located, otherwise specify the path)

`docker build -t webserver:latest .` #Build image adding a name/tag with option -t => image_name:tag

`docker push webserver:latest` #Push image to the registry

`docker images` #List images

### Containers

`docker container create <image>` #Create container (create fresh container, but doesn't run it immediately)

`docker container start <id>` #Start a container

`docker run -d --name mycontainer <image>` #Create and start a container (-d: container runs in the background --name to specify a name)

`sudo docker run -d -p 1234:80 --name website docker.io/library/nginx` #Map port 1234 on the host to port 80 on the container

`docker container ls -a`  #List containers (-a to show all, including those not running)

`docker ps --all` #Another command to list containers

`docker stop <container>` #Stop container

`docker rm <container>` #Remove container

### Logs

`docker logs <container_id>` #Show logs of a running or storred container

`docker logs -f <container_id>` #Follow logs, live stream

`docker logs --tail 15 <container_id>` #Output 15 last lines logs

## Virtual Machines

**Installation**

`sudo dnf install libvirt qemu-kvm virt-manager`

**Using a Template**

testmachine.xml

```xml
<domain type='qemu'>
  <name>Test Machine</name>
  <memory unit='GiB'>1</memory>
  <currentMemory unit='KiB'>4194304</currentMemory>
  <vcpu>1</vcpu>
  <os>
    <type arch='x86_64'>hvm</type>
  </os>
</domain>
```
**Using virt-install to Generate One**

```sh
virt-install \
  --name my-vm \
  --memory 2048 \
  --vcpus 2 \
  --disk path=/path/to/your/image.img,format=qcow2 \
  --os-type linux \
  --os-variant ubuntu20.04 \
  --network network=default \
  --graphics none \
  --import \
  --print-xml > my-vm.xml
```

**Retrieve the .xml Config of a VM**

`sudo virsh dumpxml VM1 > VM1.xml`

**Create/Start/Stop/Delete the VM**

`virsh define testmachine.xml` #Create VM with template

`virsh start TestMachine` #Start VM

`virsh autostart TestMachine` #The VM will boot automatically on PC/server startup

`virsh autostart --disable TestMachine` #Disable auto start

`virsh reboot TestMachine` #'clean' reboot

`virsh shutdown TestMachine` #'clean' shutdown

`virsh reset TestMachine` #Hard reboot

`virsh destroy TestMachine` #Hard shutdown

`virsh undefine --remove-all-storage TestMachine` #Remove VM and associated storage

**Other Options**

`virsh list` #See active VMs (--all to see all)

`virsh dominfo TestMachine`  #View info about the VM

`virsh help undefine` #Help on a specific option

`virsh console ubuntu01` #Reconnect to the VM

**Change VM Specs (RAM, CPU)**

`virsh setvcpus TestMachine 2 --config --max` #Add 2 vCPUs

`virsh setmem VM2 80M --config` #Change max mem size

`virsh setmaxmem TestMachine 2048M --config` #Change mem size

`sudo virsh shutdown VM2` + `sudo virsh start VM2` #To apply changes

**Install an Image in .img Format**

`qemu-img info <image.img>` #Inspect image

`mv image.img /var/lib/libvirt/images/` #Move the image to the directory

`sudo virt-install --import --memory 512 --vcpus 1 --disk /var/lib/libvirt/images/image.img  --osinfo debian9 --graphics none` #Install image (--import skips the installation phase)