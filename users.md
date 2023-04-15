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
file /etc/default/useradd apply to useradd only <br />
 
*SKEL:/etc/skel	*	(files in /etc/skel are created to the user home directory upon creation) <br />

alternatively write defaults to */etc/login.defs* <br />

changing these settings only affects to new created users <br />

*/etc/passwd* is used to store user properties <br />
the linux OS only cares about the uids not the name behind the uid <br />
nobody user has a low permission  <br />

*/etc/shadow* stores the passwords in a hashed way <br />

*student:$543532342:17912:0:99999:7:::* <br />
17912 -> days after 1.jan.1970 the password was set for the first time  <br />
0: minimum days the password can be used <br />
99999: max days the password can be used  <br />
7: number of days when the user gets a warning that the password expires <br />

password that is !! is currently disabled. This is the default for newly created users <br />
If the password is ! that means the account is disabled <br />

*/etc/group* is used for group policies <br />
*wheel:x:10:student* <br />

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
basic password requirements are stored in */etc/login.defs* <br />

for advanced password properties, pluggable authentication modules (PAM) can be used  <br />
&nbsp;&nbsp;&nbsp;look for the *pam_tally2* module <br />

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
 *drwx------.  3 anna   anna     78 Jun 14 08:02 anna* <br />
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
chmod -x,o+r myfile
```
### umask
the umask is a shell setting that subtracts the umask from the default permissions
default permissions for files are 666
default permissions for directories are 777

```
umask
touch root1 
ls -l root1
umask 027
touch root2
ls -l root2
mkdir rootdir
ls -ld rootdir
```
for changing the default umask change the settings in the following files:
vim /etc/default
vim ~/.bash_profile

## special permissions

| | files | directory |
| :----: | :----: | :----: |
| suid (4) | run as owner (only on executables) | - |
| sgid (2) | run as group owner | all files inheret directory group owner |
| sticky bit (1) | - | delete only if you are the owner of file or directory  | 


suid:
```
chmod 4770 myfile
chmod u+s myfile

find / -perm /4000 2> /dev/null
ls -l /usr/bin/passwd /etc/shadow
```

sgid:
```
chmod 2770 mydir
chmod g+s mydir

mkdir -p /data/profs
chown :profs /data/profs
ls -l /data/
chmod 770 /data/profs
lid -g profs
chmod g+s /data/profs/
su - audrey
cd /data/profs
touch audrey1
```

sticky bit
```
chmod 1770 mydir
chmod +t mydir

chmod +t /data/profs
```

## ACLs
ACLs are used to grant permissions to additional users and groups
normal ACL applies to existing files only
default ACL on a dirctory if you want it to apply to new files also
```
getfacl shows current settings
setfacl -R -m g:somegroup:rx /data/groups
setfacl -m d:g:somegroup:rx /data/groups
```

```
mkdir /data
cd /data
mkdir account sales
groupadd account
groupadd sales
chgrp sales sales/
chmod 770 sales/

getfacl sales/
setfacl -m d:g:account:rx sales/
ll
cd sales/
touch newfile
getfacl newfile
```

### Troubleshooting permissions
```
# cd /home/linda
# touch root1
# touch root2
# su - linda
$ ls -ld .
$ ls -l root*
$ rm -f root1                       (removing is possible for user linda, because the directory permissions apply)
$ echo "hello" >> root2             (linda can not write)
$ vim hello2                        (vim created a new file because it could not write to the original one)
$ ls -l 
$ exit 
# touch randomfile
# chown linda:linda randomfile
# chmod 006 randomfile
# su - linda
$ cat randomfile                    (not possible, because in Linux there is an exit on match algorithm)
$ chmod 666 randomfile              (as owner of a file it is possible to change the permissions)
$ exit
# echo "rootcontents" > rootfile
# chmod +t .
# ls -ld .
# su - linda
$ rm -f rootfile                    (is possible, because sticky bits allows only the owner of the file and directory to delete the file!)
```