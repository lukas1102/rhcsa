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

`su -` 	opens a new login shell, instead of a subshell <br />
*this will give complete access to the environment variables of the user*

### sudo
sudo is used to run tasks as an administrator <br />
sudo prompts the password of the current user <br />
the user needs to be authorized to use sudo <br />
/etc/sudoers and /etc/sudoers.d/* files are used for configuring sudo <br />
DO NOT directly edit these files!  -> use `visudo` <br />

```
%wheel  ALL=(ALL)       ALL			all users member of group wheel can do all commands on all computers
%users  localhost=/sbin/shutdown -h now			allows members of group users to use the shutdown command 
linda 	ALL=/usr/sbin/useradd, /usr/bin/passwd
id 			
usermod -aG wheel student
id student
```
after changing the groups of a user, the user needs to login again

## ssh
SSH is secure shell and used to establish a secure remote connection <br />
the identity of the target server is verified through the host keys <br />
&nbsp;&nbsp;&nbsp;After initial connection, host key is stored in ~/.ssh/known_hosts <br />

`ssh -X or ssh -Y` to display graphical screens from the target server locally <br />
-X is the old version and does not provide secure connection <br />
`ssh linda@localhost`

# Managing users and groups
a user is a security principle, user accounts are used to provide people or processes access to system resources <br />
process are using system accounts, which have a low uid <br />
people are using regular user accounts, which have a high uid <br />

## user properties
student:x:1000:1000:student:/home/student:/bin/bash <br />
Name: name of the account <br />
password: the secret placeholder , because it is stored in the /etc/shadow <br />
uid: unique identifier <br />
gid: id of the primary group <br />
GECOS: non mandatory information about the user <br />
home directory: environment where users create personal file <br />
shell: the program that will start after successful authentication <br />

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
`useradd -D` to specify default settings <br />
file /etc/default/useradd aplly to useradd only <br />
 
SKEL:/etc/skel		(files in /etc/skel are created to the user home directory upon creation) <br />

alternatively write defaults to /etc/login.defs <br />

changing these settings only affects to new created users <br />

/etc/passwd is used to store user properties <br />
the linux OS only cares about the uids not the name behind the uid <br />
nobody user has a low permission  <br />

/etc/shadow stores the passwords in a hashed way <br />

student:$543532342:17912:0:99999:7::: <br />
17912 -> days after 1.jan.1970 the password was set for the first time  <br />
0: minimum days the password can be used <br />
99999: max days the password can be used  <br />
7: number of days when the user gets a warning that the password expires <br />

password that is !! is currently disabled. This is the default for newly created users <br />
If the password is ! that means the account is disabled <br />

/etc/group is used for group policies <br />
wheel:x:10:student <br />

## group membership
each user has at least 1 group (primary group) <br />
the primary group membership is managed through /etc/passwd <br />
the user primary group becomes group-owner if a user creates a file <br />
additional groups (secondary) can be defined, these are managed by the /etc/group <br />

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
basic password requirements are stored in /etc/login.defs <br />

for advanced password properties, pluggable authentication modules (PAM) can be used  <br />
&nbsp;&nbsp;&nbsp;look for the pam_tally2 module <br />

```
chage student
grep student /etc/group
chage --help |less
```

# managing permissions
## understanding ownership
 to determine which permissions a user has, Linux uses ownership <br />
 every file has a user-owner, a group-owner and the others entity that is also granted permissions (ugo) <br />
 linux permissions are not additive, if you are the owner, permissions are applied and that´s all <br />
 
 `ls -l` <br />
 drwx------.  3 anna   anna     78 Jun 14 08:02 anna <br />
 d -> directory  <br />
 rwx -> owner permissions <br />
 owner -> anna <br />
 group -> anna <br />
 
 ### changing file ownership
 chown user[:group] file to set user-ownership <br />
 chgrp group file to set group-ownership <br />
 
 ```
 touch newfile
 ls -l newfile
 chown anna newfile
 chown anna:profs newfile
 chgrp students newfile
 ```
 
 ## basic permissions
 
| | files | directory |
| :----: | :----: | :----: |
| read | read | list |
| write | modify | delete/create |
| execute | run | cd | 

execute always comes with read <br />

`chmod` is used to manage permissions <br />

```
chmod 750 myfile
chmod +x myscript

```