# basics in networking
- in ipv4 each node needs its own ip address, written in dotted decimal notation (192.168.0.1/24)
- each ip address must be inidcated with the subnet mask
- the default router or gateway specifies which server to forward packets to the outside world
- the DNS nameserver is the ip address of a server that helps resolving names to ip addresses
- ipv4 is most common IP, but ipv6 address can be used as well
- ipv6 addresses are written in hexadecimal notation (fd01::8eba:210)
- ipv4 and ipv6 can co-exist on the same network interface

## NIC (network interface card) naming
- ip address configuration needs to be connected to specific network device
- use `ip link show` to see the current devices and `ip addr show` to check their configuration
- every system has an lo device (loopback), which is for internal networking
- apart from that, you'll see the name of the real network device, which is presented as a BIOS name
- classical naming is using device names like eth0, eth1 and so on
-- These device names don't reveal any information about physical device location
- BIOS naming is based on hardware properties to give more specific information in the device name
-- em[1-N] for embedded NICs
-- eno[nn] for embedded NICs
-- p<slot>p<port> for NICs on the PCI bus
- if the driver doesn't reveal network device properties, classical naming is used 

## managing runtime configuration with ip
All these configuration is available during runtime, it does not save anything
- the ip tool can be used to manage all aspects of IP networking
- it replaces the `ifconfig` tool, do NOT use `ifconfig` anymore
- use `ip addr` to manage address properties
- use `ip link` to show link properties
- use `ip route` to manage route properties
```
ip link show
ip -s link show

ip addr show
ip addr add dev ens33 10.0.0.10/24
ip a s
ip route show
ip route del default via 192.168.4.2
ip route add default via 192.168.4.2
cat /etc/resolv.conf
```
## understanding RHEL 8 networking

nmcli  nmtui
 |       |
Network Manager
      |
/etc/sysconfig/network-scripts/ifcfg-ens33   --> runtime (ip)
      |
    ens33

## managing persistent networking with nmcli
- an nmcli connection is a configuration that is added to network device
- connections are stored in configuration files
- the networkManager service must be running to manage files
- ensure that the bash-completion RPM package is installed when working with nmcli

```
systemctl status NetworkManager
rpm -qa | grep bash-completion
man nmcli-examples
nmcli connection add ifname ens33 ipv4.addresses 192.168.4.208/24 ipv4.gateway 192.168.4.2 \
ipv4.method manual ipv4.dns 8.8.8.8 type ethernet 

nmcli connection show
nmcli connection up ethernet-ens33
nmcli connection show
ip a s
```
## verifying network configuration files
```
cd /etc/sysconfig/network-scripts
ls
cat ifcfg-ethernet-ens33            (onboot=yes bring network up after reboot, bootproto=none is for disabling dhcp)
```

## testing network connections
- `ping` is uded to test connectivity
- `ip addr show` shows current configuration
- `ip route show` shows current routing table
- `dig` can test DNS namserver working
```
ping google.com
ping -c 1 google.com
ping -f google.com

dig rhatcert.com
dig werterffsf.be
```
