# Setting up a Base Server
- On a VM with a disk with a size of at least 20GiB, install a Contos or RHEL 8 server, ensure that after installation a graphical interface is started automatically
- Use custom partitioning and create the following partitions
- - 1GB /boot
- - 12 GB /
- - 1 GB swap
- Assume that you have lost access to the root account and cannot login in as user root anymore. Reset the root password
- Loop mount the installation disk that you've used to set up RHEL 8. Configure the loop-mounted ISO as a repository
- Confiugre your system to us this repository as the only repository
- In the remaining disk space on your server hard disk, add a 1GiB partition. Do this in such a way that it's possible to add more partitions later
- Format this partition with the Ext4 file system and set the label dbfiles on the partitions. Confiugre your system to mount this partition persistently on the directory /dbfiles, using the partition label
- Create a 2GiB LVM volume group with the name vgdata
- In this volume group, create a 1GiB logical volume with the name lvdata
- Format this logical volume with the XFS file system and mount it persistently on the directory /lvdata
- Restart your computer, and after a restart add another 500 MiB to the XFS file system that was created on top of this logical volume
- Set passwords for users to expire after 90 days. Also ensure that new passwords must have a length of at least 6 characters
- Ensure that while creating new users, an empty file with the name newfile is created in their home directories
- Create users victoria and karen Make them a membership of the group students as a secondary group membership
- Create users anna and amy. Make them a member of the group profs as a seondary group membership
- Create shared group directoriess /data/students and /data/profs and ensure that members of the group students have full access to /data/students, and members of profs have full access to /data/profs. The others entity should have no access at all
- Ensure that all new files in these directories are automatically group-owned by the group-owner of the directory
- Ensure that only the owner of a file is allowed to remove files
- User anna is head-master and should be allowed to remove all files
- All users from the group profs should have read permissions on all files in /data/students
- Schedule a cron job to automatically wirte the text "hello world" to syslog at every 10th minute after the hour. Ensure this message is written with the "notice" priority
- Ensure that the registry.access.redhat.com container repository is used 
- Download th nginx image to the local computer
- Start the nginx container, meeting the following requirements
- - the container should have a writeable directory /var/lib/nginx that is bind mounted to the directory /webfiles on th host operating system
- - The container is accessible on port 8080 on the host, which is mapped to port 80 within the container
- Configure your system such that the container is started automatically as a user systemd service

# resetting the root password
reboot
e + rd.break
mount
ctrl + d
ctrl + d

# configuring repositories
df -h
dd if=/dev/sr0 of=/rhel8.iso bs=1M

# Managing partitions
lsblk
parted /dev/nvme0n1
print
quit
fdisk /dev/nvme0n1
p
n
e
n
w
partprobe
reboot
lsblk
mkfs.ext4 -L dbfiles /dev/nvme0n1p5
mkdir /dbfiles
echo "LABEL=dbfiles     /dbfiles        ext4    defaults    0 0" >> /etc/fstab
mount -a
mount

# Managing LVM 
fdisk /dev/nvme0n1
n
+2G
t
8e
p
w
lsblk
vgcreate vgdata /dev/nvme0n1p6
vgs
lvcreate -n lvdata -L 1G vgdata
lvs
mkfs.xfs /dev/vgdata/lvdata
mkdir /lvdata
echo "/dev/vgdata/lvdata    /lvdata     xfs     defaults    0 0" > /etc/fstab
reboot
df -h
vgs
lvextend -r -L +500M /dev/vgdata/lvdata
df -h

# Creating user groups 
vim /etc/login.defs
touch /etc/skel/newfile
groupadd students
groupadd profs
useradd -G students victoria
useradd -G profs anna
id anna

# Managing permissions
mkdir -p /data/students /data/profs
cd /data
ls -l
chmod 3770 *
ls -l
chown anna:students students/
chown anna:profs  profs/
ls -l
setfacl -m g:profs:rx /data/students
setfacl -m d:g:profs:rx /data/students
getfacl
su - victoria
touch /data/student/file1
getfacl /data/student/file1

# Scheduling Jobs Solution
logger --help
logger -p notice hello
grep hello /var/log/messages
crontab -e
10 * * * * logger -p notice hello world

# Managing Containers as Service
sudo useradd linda
sudo passwd linda
firewall-cmd --add-port 8080/tcp --permanent
firewall-cmd --reload
sudo mkdir /webfiles
sudo chown linda /webfiles
sudo loginctl enable-linger linda
as linda: podman login
podman pull nginx
podman run -d -v /webfiles:/var/lib/nginx:Z -p 8080:80 --name mynginx nginx
mkdir ~/.config/systemd/user
podman generate systemd --name mynginx --files
systemctl --user daemon-reload
systemctl --user enable container-mynginx.service

reboot