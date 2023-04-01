# Understanding Automated Installations Solutions
Several solutions exist for performing automated installations
- vagrant is used for automatic deployment of virtual machines
- cloud-init and other templates can be used in cloud environments
- Kickstart can be used with a PXE-boot server to provide instructions for automatic installation of RHEL
A kickstart file contains all instructions to set up a RHEL instance. It can be used to easily reproduce installations

# Creating a Kickstart file
- After installation, a file anaconda-ks.cfg is created to the root user home directory
- Edit this file manually to make any changes that are required

```
ls -l /root/*cfg
vim initial-setup-ks.cfg
```

# Using the kickstart file for automatic installation
- Typically, the kickstart file is provided on an installation server
- Before starting the installation, the client indicates where to get the kickstart file from
- - use `ks=http://somewhere/ks.cfg`
- - or provide interface provided by the installation program (as in virtual machine manager)

On the installation prompt, press the tab-key to show the kernel boot parameter
here you can append the kickstart line like `ks=http://myserver/ks.cfg`

# Using Kickstart files in fully automated datacenters

                        -> access repos
start installation      give kickstart <- 
                        -> get kickstart        http://ks.cfg
install loads           boot image  < -             tftp
                                                    dhcp
pxe boot                        ->              install server 

# Using Vagrant to set up virtual machines
- vagrant is a solution to automate installing virtual machines
- vagrant works with a "box", which is a tar file that contains a VM image
- Preconfigured boxes are available at vagrantcloud.com
- Administrators can create their own boxes
- Providers allow vagrant to interface with the underlying host platform
- - supported platforms are virtualbox, vmware, hyper-vm and KVM
- Provisioners can be used to further configure a vagrant-configured VM
- - Bash and Ansible are common provisioners
- The vagrantfile is a text file containing the constructions for creating the Vagrant envrionment
- Vagrant is not included in RHEL 8 and must be installed from EPEL
