# Understanding the Boot procedure
    services
    baseos
    systemd manager
    Kernel, initramfs (temp root fs and basic drivers)
    GRUB bootloader
UEFI    BIOS
POST  (power on self test)

# Modifying Grub2 runtime parameters
- from the Grub2 boot menu, press e to edit runtime boot options
- Press c to enter the Grub2 command mode
- - from command mode, type help for an overview of available options

Press the arrow keys within 5 sec, otherwise the system will boot
the last 3 kernels will kept around.
Normally there are the Kernel and the rescue kernel (boots with very restrictive options)

ctrl + e
linux line is loading the kernel 
Options: rhgb (redhat grafical boot) quiet (don´t show me anything)
ctrl + x to start

# Modifying Grub2 persistent parameters
- to edit persistent Grub2 parameter, edit the configuration file in /etc/default/grub
- After writing changes, compile changes to grub.cfg
- - `grub2-mkconfig -o /boot/grub2/grub.cfg`
- - `grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg`

```
mount | grep '^/'
mount | grep '^/' | grep -i efi
vim /etc/default/grub
grub2-mkconfig -o /boot/grub2/grub.cfg
vim /boot/grub2/grub.cfg
reboot
```

# Managing Systemd Targets
- A systemd target is a group of unit files
- Some targets are isolateable, which menas that they define the final state a system is starting in
- - emergency.target
- - rescue.target
- - multi-user.target
- - graphical.target
- When enabling a unit, it is added to a specific target

```
systemctl disable httpd
systemctl enable httpd
systemctl get httpd
systemctl cat multi-user.target
cd /etc/systemd/system
ls
cd multi-user.target.wants
ls
systemctl list-dependencies
```

# Setting the Default Systemd target
 - Use `systemctl get-default` to see the current default target
 - Use `systemctl set-default` to set a new default target

 ```
 systemctl get-default
 systemctl set-default multi-user.target
 reboot
 systemctl set-default graphical.target
 systemctl start graphical.target
 ```

 # Booting into a Specific target
 - On the Grub2 boot prompt, use systemctl.unit=xxx.target to boot into a specific target
 - To change between targets on a running system, use systemctl isolate xxx.target

 ```
 systemctl list-units
 systemctl isolate emergency.target
 systemctl list-units
 systemctl start graphical.target
 ```
