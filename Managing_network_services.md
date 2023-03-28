# Understanding SSH Key-based Login
ssh-keygen
- private
- public        -> ssh-copy-id      -> ~ pub

## setting up ssh key-based login
- `ssh-keygen` creates a public/private key pair for the current user
- - setting a passphrase for the private key makes it more secure, but less convenient
- `ssh-copy-id` copies the public key over to the target server
- `ssh-agent /bin/bash` allocates space in the bash shell to cache the private key passphrase
- `ssh-add` adds the current passphrase to the cache

```
ssh-keygen
ssh-copy-id 192.168.4.237
ssh 192.168.4.237
exit
ssh-agent /bin/bash
ssh-add
ssh 192.168.4.237
```

## Changing common SSH server settings
- Server options are set in /etc/ssh/sshd_config
- Client options can be set in /etc/ssh/ssh_config
- - Port 22
- - PermitRootLogin no
- - PubkeyAuthentication yes
- - PasswordAuthentication yes
- - X11Forwarding yes
- - AllowUsers student

systemctl restart sshd
ssh root@localhost

## Securely copying files
- `scp` can be used to securely copy files over the network, using the sshd process
- - `scp file1 file2 student@remoteserver:/home/student`
- - `scp -r root@remoteserver:/tmp/files .`
- `sftp` offers an FTP client interface to securely transfer files using SSH
- - Use `put /my/file/` to upload a file
- - Use `get /your/file` to download a file to the current directory
- - Use `exit` to close an sftp session

```
scp /etc/hosts linda@192.168.4.237:/tmp/
mkdir temp
scp -r root@192.168.4.237:/etc temp/
ls temp
sftp 192.168.4.237
help
lpwd
pwd
cd /
pwd
ls
cd etc
lpwd
lcd /tmp
lpwd
get /etc/passwd
put /etc/passwd
exit
```

## Securely synchronizing files
- `rsync` is using SSH to synchronize files
- If source and target file already exists, `rsync` will only synchronize their differences
- The `rsync` command can be used with many options, of which the following are most common
- - `-r` will recursively synchronize the entire directory tree
- - `-l` synchronizes symbolic links
- - `-p` preserves symbolic links
- - `-n` will do a dry run before actually synchronizing
- - `-a` uses archive mode, which is equivalent to -rlptgoD
- - `-A` uses archive mode, and also synchronizes ACLs
- - `-X` will synchronize SELinux context as well

```
ssh 192.168.4.237
touch /etc/mynewfile{1..10}
exit
rsync -ar root@192.168.4.237:/etc/ temp/
ls temp/mynewfile*
```

# Understanding Apache Configuration
- Apache (httpd) is aleading web server on Linux
- Nginx is the other leading web server
- The main httpd configuration file is /etc/httpd/conf/httpd.conf
- Additional snap-in files can be stored in /etc/http/conf.d/
- The default DocumentRoot is /var/www/htdocs
- Apache looks for a file with the name index.html in this directory

```
yum install httpd
systemctl enable --now httpd
systemctl status httpd
yum install -y curl
curl http://localhost
less /etc/httpd/conf/httpd.conf
```

## Creating a basic website
```
cd /var/www/html
ls
echo "hello world" > index.html
systemctl restart httpd
curl http://localhost
```
