# Understanding containers
- A container is a complete package to run an application, which contains all application dependencies
- Containers make it easier to run different versions of application dependencies side-by-side
- Containers run on top of a container engine that is offered by the host operating system
- The operating system kernel is not included in the container, but offered by the host
- By using this approach, containers are more efficient than virtual machines
- To start a container, container images are used
- The purpose of a container is to run an isolated process
- To do this, container images are configured to run a standard application
- Once that application is finished, the container is done
- Application containers are used to run common applications
- System containers are used as the foundation to build custom images, and do not come with a standard application
- Containers are Linux, and rely heavily on features provided by the Linux operating system
- - Namespaces for isolation between processes
- - Control Group for resource management
- - SELinux to ensure security
- One container runs one application
- Multiple applications can be connected in Microservices
- To manage containers at an enterprise level, orchestration is needed
- RedHat has OpenShift ot provide orchestration
- Different solutions for managing contaers exist
- These solutions are highly compatible and interchangable because of the Open Containers Initiative
- On previous versions of RHEL, Red Hat was offering Docker support
- In RHEL 8, RedHat is offering some new utilities
- - `podman` is used to manage containers and container images
- - `buildah` is used to create new container images
- - `skopeo` is used to inspect, delete, copy and sign images
- If containers need to run a process on a privileged port, they need to run with root privileges
- Rootless containers will run as a non-root user
- Running containers as root is more dangerous
- Running containers in an enterprise environment requires a solution that offers additional features
- - Easily scale containers up and down
- - Ensure container availability
- - Offer a developer workflow to make it easy to get from source code to running container
- Kubernetes is the common orchestration standard that is used to manage containers in a cluster environment
- Red Hat OpenShift is the Red Hat Kubernetes distribution; it offers additional features to Kubernetes

# Running a Container
- To start with, container management tools need to be installed: `yum module install container-tools`
- After installing, you can immediately start running containers from the Docker registry: `podman run -d nginx`
- The Red Hat registries are registry.redhat.io for official Red Hat products, and registry.connect.redhat.com for third-party products
- Red Hat registry access requires authentication using `podman login`, where the Red Hat account name and password are provided
- Registries are processed in order as in /etc/containers/registries.conf
- To get a specific container, use a complete name reference: `podman pull registry.access.redhat.com/ubi8/ubi:latest`
- Use `podman pull` to pre-pull the image from the registry to the local system
- Use `podman run` to pull the container (if necessary) and run it
- - Will run the container in the foreground
- - Use `podman run -d` to run in detached mode
- - Use `podman run -it` to run in interactive tty mode
- - Consider using the option `--rm` to remove the container after using it
- Detach from a container tty using `ctrl-p, ctrl-q`
- Exit from the primary container application using `exit`

```
$ podman run nginx
^c
podman run -d nginx
podman ps
podman ps --all
$ podman run ubuntu
$ podman ps
$ podman ps --all
$ podman run -it ubuntu
  ps aux
  ls
  ip a
  cat /etc/os-release
  uname -r
  ctrl-p, ctrl-q
docker ps
uname -r
podman login
podman pull registry.access.redhat.com/ubi8/ubi:latest
```

# Managing Images
- The image is the read-only runnable instance of the container
- Images are obtained from registries, which are specified in /etc/containers/registries.conf
- Additional registries can be added in the [registries.search] section in this file
- Use `podman info` to see which registries are currently used
- Insecure registries are not protected with TLS encrytion and must be listed in [registries.insecure]
- `podman search` searches all registries
- `podman search --no-trunc registry.redhat.io/rhel8` searches specific registry on the rhel8 string
- Use filter to further limit the search result
- - `--limit 5` shows a maximum of 5 images per registry
- - `--filter stars=5` shows images with 5 stars or more
- - `--filter is-official=true` shows official images only
- Web search is available through https://access.redhat.com/containers or https://hub.docker.io
- Use `skopeo` to inspect images before pulling them
- `skopeo inspect docker://registry.redhat.io/ubi8/ubi`
- Use `podman` to inspect images that are locally available
- - `podman images`
- - `podman inspect registry.redhat.io/ubi8/ubi`
- Notice that some Docker images are designed to run as root, they won't run in podman without `sudo`
- Use complete URLs to the image you want to install to increase your chance of being successful
- - `podman run -d registry.access.redhat.com/rhscl/httpd-24-rhel7`
- When newer versions of images become available, the old version will be kept on your system as well
- Use `podman images` to get a list of all images
- Use `podman rmi` to remove images
- For advanced image management the `buildah` utility is provided
- This tool can be used to create your own custom images and offers different ways to do so:
- - Based on a Dockerfile with `buildah bud`
- - By running `buildah` commands directly against an image using `buildah run`

```
$ sudo vim /etc/containers/registries.conf
$ podman info | less
$ podman search ubuntu
$ podman search --no-trunc registry.redhat.io/ubi8
$ podman search --no-trunc registry.redhat.io/ubi8 --filter stars=5
$ podman search --no-trunc registry.redhat.io/ubi8 --limit 5
$ podman images
$ podman inspect registry.access.redhat.com/ubi8/ubi
$ podman rmi docker.io/library/mariadb
$ podman ps -a
```

# Managing Containers
- Map a host port to the container application port to make it reachable from the outside
- `podman run -d -p 8080:80 nginx` will map host port 8000 to container port 80
- `podman port -a` will show all current container port mappings
- Do not forget to open these ports in the host firewall: `firewall-cmd --add-port=8000/tcp [permanent]`
- Containers running without root privileges can bind to a non-privileged host port only
- Some containers require environment variables to run them
- If a container fails, because of this requirement, use `podman logs containername`, it shows the application log telling you why it failed
- Alternatively, use `podman inspect` and look for a `usage` line
- Use `-e VAR=value` while starting the container to pass variable values: `podman run -d --name mydb -e MYSQL_ROOT_PASSWORD=password -e MYSQL_USER=bob -e MYSQL_PASSWORD=password -e MYSQL_DATABASE=books -p 3306:3306 mariadb`
- `podman ps` shows currently running containers
- `podman ps -a` shows containers in a stopped state
- `podman stop mycontainers` stops a container gracefully sending SIGTERM, if that does not work after 10 sec the container receives SIGKILL
- `podman kill mycontainers` sends SIGKILL to the containers
- `podman rm mycontainers` removes a container, including all file modifications written to the container writable layer
- `podman restart mycontainer` restarts a container that was previously stopped
- `podman exec mycontainer uname -r` runs an additional process inside a running container
- `podman exec -it mycontainer /bin/bash` accesses an interactive shell
- `podman exec -l cat /etc/redhat-release` runs the command on the last container that was used in any command
- By default, podman runs non-root containers
- Non-root containers cannot bind to a privileged port and do NOT have an IP address
- The only way to acccess their application is by using port forwarding
- If you need a container that has an IP address, you need to run it as a root container: `sudo podman run -d nginx`
- Root containers will only show if you use `sudo podman ps`
- After starting a root container, on the host operating system a bridge device is created that works like NAT to make sure the root container can access the external network

```
$ podman ps
$ podman run -d -p 8000:80 nginx
$ podman port -a
$ sudo firewall-cmd --add-port=8000/tcp --permanent
$ sudo firewall-cmd --add-port=8000/tcp
$ podman run mariadb
$ podman ps -a
$ podman logs practical_chaum
$ podman run -d --name mydb -e MYSQL_ROOT_PASSWORD=password -e MYSQL_USER=anna -e MYSQL_PASSWORD=password -e MYSQL_DATABASE=trainers -p 3306:3306 mariadb
$ podman logs mydb
sudo firewall-cmd --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
$ podman exec -it mydb /bin/bash
    ps aux
```

# Attaching Storage to Containers
- Container storage is ephermeral
- Modifications are written to the container writeable layer and stay around for the container lifetime
- Persistent storage keeps files externally
- Use bind mounts to connect a directory inside the container to a directory on the host
- Ensure that the user account used in the container has access to the host directory, and set the SELinux context type to `container_file_t`
- If the container user is owner of the host directory, the :Z option can be used: `podman run -d -v /webfiles:/webfiles:Z nginx`
- Using SELinux is essential, without it all root containers would have full access to the host filesystem!
- `sudo mkdir /dbfiles`
- `sudo chmod o+w /dbfiles`
- `sudo semanage fcontext -a -t container_file_t "/dbfiles(/.*)?"`
- `podman run -d --name ymdb -v /dbfiles:/var/lib/mysql:Z -e MYSQL_ROOT_PASSWORD=password -e MYSQL_USER=bob -e MYSQL_PASSWORD=password -e MYSQL_DATABASE=books -p 3306:3306 mariadb`

```
$ sudo mkdir /dbfiles
$ sudo chmod o+w /dbfiles
$ podman run -d --name mynewdb -v /dbfiles:/var/lib/mysql:Z -e MYSQL_ROOT_PASSWORD=password mariadb
$ podman ps
$ podman logs mynewdb
sudo grep AVC /var/log/audit/audit.log
sudo chown student:student /dbfiles
$ podman run -d --name mynewdb -v /dbfiles:/var/lib/mysql:Z -e MYSQL_ROOT_PASSWORD=password mariadb
$ podman rm mynewdb
$ podman run -d --name mynewdb -v /dbfiles:/var/lib/mysql:Z -e MYSQL_ROOT_PASSWORD=password mariadb
 podman ps
 podman exec mynewdb ls /var/lib/mysql
```

# Managing Containers as Services
- To automatically start containers in a stand-alone situation, you can create systemd user unit files for rootless containers and manage them with `systemctl`
- If Kubernetes or OpenShift is used, containers will be automatically started by default
- Systemd user services start when a user session is opened, and close when the user session is stopped
- Use `loginctl enable-linger` to change that behaviour and start user services for a specific user (requires root privileges)
- - `loginctl enable-linger linda`
- - `loginctl show-user linda`
- - `loginctl disble-linger linda`
- Create a regular user account to manage all containers
- Use `podman` to generate a user systemd file for an existing container
- Notice the file will be generated in the current directory
- - `podman generate systemd --name myweb --files`
- To have systemd create the container when the service starts, and delete it again when the service stops, add `--new`
- - `podman generate systemd --name ephermeral_ellie --files --new`
- To generate a service file for a root container, do it from /etc/systemd/system as the current directory
- Create user specific unit files in ~/.config/systemd/user
- Manage them using `systemctl --user`
- `systemctl --user daemon-reload`
- `systemctl --user enable myapp.service` (requires linger)
- `systemctl --user start myapp.service`
- `systemctl --user` commands only work when logging in on console or SSH and do not work in sudo and su sessions

```
$ sudo -i
# loginctl enable-linger student
# loginctl show-user student
$ mkdir -p .config/systemd/user
$ cd .config/systemd/user
$ podman ps
$ podman generate systemd mynewdb --files
$ podman generate systemd --name mynewdb --files
$ systemctl --user daemon-reload
$ systemctl --user enable container-mynewdb.service
$ systemctl --user status container-mynewdb.service
# reboot
$ podman ps
$ systemctl --user status container-mynewdb.service
```