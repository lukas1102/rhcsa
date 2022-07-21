## Connecting to RHEL 
### Understanding the root User

users processes <br />

user space <br />
----------- (permissions, syscall) <br />
kernel space	(root) <br />
drivers <br />
hardware <br />

### virtual terminal
/dev/tty (device files)
tty1 - tty6 are virtual terminals
use `chvt` to switch between virtual terminals
or use ctrl + Alt - Fn 
```
chvt 4
w
```

### su
su is for switching the user
the root user can switch to any user without entering the password
as normal user you need the password of the target user

`su -` 	opens a new login shell, instead of a subshell
*this will give complete access to the environment variables of the user*

### sudo
sudo is used to run tasks as an administrator
sudo prompts the password of the current user
the user needs to be authorized to use sudo
/etc/sudoers and /etc/sudoers.d/* files are used for configuring sudo
DO NOT directly edit these files!  -> use `visudo`

```
%wheel  ALL=(ALL)       ALL			all users member of group wheel can do all commands on all computers
%users  localhost=/sbin/shutdown -h now			allows members of group users to use the shutdown command
linda 	ALL=/usr/sbin/useradd, /usr/bin/passwd
id 			
usermod -aG wheel student
id student
```
*after changing the groups of a user, the user needs to login again*

## ssh
*SSH is secure shell and used to establish a secure remote connection*
*the identity of the target server is verified through the host keys*
&nbsp;&nbsp;&nbsp;*After initial connection, host key is stored in ~/.ssh/known_hosts*

`ssh -X or ssh -Y` to display graphical screens from the target server locally
-X is the old version and does not provide secure connection
`ssh linda@localhost`

# Managing users and groups
a user is a security principle, user accounts are used to provide people or processes access to system resources
process are using system accounts, which have a low uid
people are using regular user accounts, which have a high uid

## user properties
student:x:1000:1000:student:/home/student:/bin/bash
Name: name of the account
password: the secret placeholder , because it is stored in the /etc/shadow
uid: unique identifier
gid: id of the primary group
GECOS: non mandatory information about the user
home directory: environment where users create personal file
shell: the program that will start after successful authentication

## creating and managing accounts

```
useradd --help |less
useradd -c bill bill 

usermod --help |less
usermod -aG wheel bill
id bill

userdel --help
userdel -f bill

passwd --help
passwd -l linda
passwd -u linda
passwd linda

```

## user default settings
`useradd -D` to specify default settings
file /etc/default/useradd aplly to useradd only

SKEL:/etc/skel		(files in /etc/skel are created to the user home directory upon creation)

alternatively write defaults to /etc/login.defs

changing these settings only affects to new created users

/etc/passwd is used to store user properties
the linux OS only cares about the uids not the name behind the uid
nobody user has a low permission 

/etc/shadow stores the passwords in a hashed way

student:$543532342:17912:0:99999:7:::
17912 -> days after 1.jan.1970 the password was set for the first time
0: minimum days the password can be used
99999: max days the password can be used 
7: number of days when the user gets a warning that the password expires

password that is !! is currently disabled. This is the default for newly created users
If the password is ! that means the account is disabled

/etc/group is used for group policies
wheel:x:10:student

## group membership
each user has at least 1 group (primary group)
the primary group membership is managed through /etc/passwd
the user primary group becomes group-owner if a user creates a file
additional groups (secondary) can be defined, these are managed by the /etc/group

```
id
grep student /etc/passwd
grep student /etc/group
```

## managing groups

```
groupadd --help |less
groupadd sales
groupmod --help |less
groupdel --help |less
groupdel sales

lid -g wheel
grep wheel /etc/group
```

## managing password properties
basic password requirements are stored in /etc/login.defs

for advanced password properties, pluggable authentication modules (PAM) can be used 
&nbsp;&nbsp;&nbsp;look for the pam_tally2 module

```
chage student
grep student /etc/group
chage --help |less
```

# managing permissions
## understanding ownership
 to determine which permissions a user has, Linux uses ownership
 every file has a user-owner, a group-owner and the others entity that is also granted permissions (ugo)
 linux permissions are not additive, if you are the owner, permissions are applied and that´s all
 
 `ls -l`
 drwx------.  3 anna   anna     78 Jun 14 08:02 anna
 d -> directory 
 rwx -> owner permissions
 owner -> anna
 group -> anna
 
 ### changing file ownership
 chown user[:group] file to set user-ownership
 chgrp group file to set group-ownership
 
 ```
 touch newfile
 ls -l newfile
 chown anna newfile
 chown anna:profs newfile
 chgrp students newfile
 ```
 
 ## basic permissions
 
| | 
files | directory |
| :----: | :----: | :----: |
| read | read | list |
| write | modify | delete/create |
| execute | run | cd | 

execute always comes with read

`chmod` is used to manage permissions

```
chmod 750 myfile
chmod +x myscript

```