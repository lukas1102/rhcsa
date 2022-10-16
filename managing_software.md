# understanding RPM packages
redhat package manager (rpm) is a package format to install software, as well as a database of installed packages on a system.
the package contains an archive of files that is compressed with cpio, as well as metadata and a list of package dependencies.
RPM packages may contain scripts as well 
to install packages repositories are used
individual packages may be installed, but this should be avoided

# setting up repository access
we'll create a local repository so that we can install packages from the RHEL 8 installation disk ISO image
- create an ISO image: `dd if=/dev/sr0 of=/rhel8.iso bs=1M`
- creat a directory /repo: `mkdir /repo`
- edit /etc/fstab and add the following line to the end:
- - /rhel8.iso /repo iso9660 defaults 0 0
- `systemctl daemon-reload`
- use `mount -a` to mount the ISO
- create the file /etc/yum.repos.d/appstream.repo with following contents
```
[appstream]
name=appstream
baseurl=file:///repo/AppStream
gpgcheck=0
```
- create the file /etc/yum.repos.d/base.repo with following contents
```
[baseOS]
name=baseOS
baseurl=file:///repo/BaseOS
gpgcheck=0
```
- yum repolist

# understanding modules and application streams
- RHEL 8 introduces application streams and modules
- application streams are used to separate user space packages from core kernel operations
- using application streams allows for working with different versions of packages
- Base packages are provided through the BaseOS repository
- AppStream is provided as a separate repository
- Application streams are delivered in two formats
- - traditional RPMs
- - new Modules
- Modules can contain streams to make multiple versions of applications available
- Enabling a module stream gives acces to RPM packages in that stream
- Modules can also have profiles. A profile is a list of packages that belong to a specific use-case (minimum installation, complete installation)
- use the `yum module` commands to manage modules

# managing packages with yum
`yum` was created to be intuitive
```
yum search  (package name and description)
yum search nmap
yum install 
yum install nmap
yum remove
yum remove nmap
yum update
yum update kernel
yum provides
yum search sepolicy
yum provides */sepolicy (in files of all packages)
yum info
yum info nmap 
yum list
yum list all
yum list installed
```
# managing modules and application streams
- the `yum module` command is used to manage module properties
- `yum module list`
- `yum modules provides httpd` searches the module that provides a specific package
- `yum module info php`
- `yum module info --profile php` shows profiles
- `yum mdoule list php` shows which streams are available
- `yum module install php:7.1` or `yum install @php7.1`
- `yum module install php:7.1/devel` installs a specific profile
- `yum install httpd` will have yum automatically enable the moudle stream this package is in before installing this package
- `yum module enable php7.1` enables the module but doesn't install anything yet
Updating module content
`yum module install php:7.1` will install a specifc PHP module stream
- - this will also enable the 7.1 stream
`yum module install php:7.2` will update to the newer version
- - the 7.1 stream will be disabled, and the 7.2 stream will be enabled
- to upgrade or downgrade packages from a previous module stream that are not listed in profiles that are installed with the module update, use `yum distro-sync`

```
yum module list
yum module info php
yum module info --profile php:7.1
yum module install php:7.1
yum update
yum module install php:7.2
yum module install php:7.1
```
# using yum groups
- `yum groups` are provided to give access to specific categories of software
- `yum groups list` gives a list of most common yum groups
- `yum groups list hidden` shows all yum groups 
- `yum groups info <groupname>` shows which packages are in a group
- `yum groups install <groupname>` will install a specific yum group

```
yum groups list
yum groups list hidden
yum groups info "Directory Client"
yum groups instll --with-optional "Directory Client"
```
# managing yum updates and yum history
- `yum history` gives a list of recently issued commands
- `yum history undo` allows you to undo a specific command, based on the histroy information
- `yum update` will update all packages on your system
- `yum update <packagename>` will update on package only, including its dependencies
```
yum history
yum history undo 3
```
# using rpm queries
- rpm is the legacy command to manage RPM packages
- Do NOT use rpm to install packages as it does not consider dependencies
- rpm is useful though to perform package queries
- rpm queries by default are against the database of installed packages, add - q to query package files
- - rpm -qf /any/file   (package name where the file is coming from)
- - rpm -ql mypackage   (shows files in a package)
- - rpm -qc mypackage   (will show configuration files)
- - rpm -qp --scripts mypackage-file.rpm    (packages that you have in a file, but not yet installed. 
                                            It will look if there are any scripts in the package mypackage-file.rpm)
```
ll /etc
rpm -qf /etc/tcsd.conf
rpm -ql trousers
rpm -qd trousers
rpm -qc trousers
man tcsd
yum install dnf-utils -y
yumdownloader httpd
ls
rpm -qp --scripts httpd*.rpm
```
# using redhat subscription manager
- to work with the RHEL repositories, you need a subscription
- if you want to evaluate, use the free developer subscription
- - get it from https://developer.redhat.com
- next, use the subscription manager to set up the subscription
- use `subscription-manager` register to register
- - username is the username of the red hat account
- - password is the associated password
- next use the `subscription-manager attach --auto` to connect your current subscription
```
subscription-manager register
subscription-manager attach --auto
yum repolist
```