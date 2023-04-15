# Understanding Linux Time
NTP     1000sec !

        timedatectl
        date
system
  |      hwclock
hardware  

# Setting time with timedatectl
- `hwclock`: set hardware clock and synchronize with system time
- `date`: set current time and display format
- `tzselect`: allows to select the current time zone
- `timedatectl` new utility to manage all aspects of time

```
timedatectl --help
timedatectl status
timedatectl list-timezones
timedatectl set-timezone America/Los_Angeles
timedatectl show
```

# Setting up an NTP client
```
ssh 192.168.4.210
vim /etc/chrony.conf
sed -i 's/^#local stratum.*/local stratum 5/' /etc/chrony.conf
sed -i 's/^#allow 192.168.*/allow 192.168.0.0\/16/' /etc/chrony.conf
sed -i 's/^pool.*/pool 2.centos.pool.ntp.org iburst/' /etc/chrony.conf
systemctl restart chronyd
systemctl status chronyd
date
firewall-cmd --list-all
firwall-cmd --add-service ntp
firewall-cmd --add-service ntp --permanent
exit

sed -i 's/^pool.*/server 192.168.4.210/' /etc/chrony.conf
systemctl restart chronyd
chronyc sources

date -s 10:07
timedatctl status

timedatectl set-ntp yes

```