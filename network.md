# 16. Networking

## Network interfaces

Network interface names start with the type of interface:

- Ethernet interfaces begin with en
- WLAN interfaces begin with wl
- WWAN interfaces begin with ww

The rest of the interface names after the type will be based on information provided by the server's firmware or determined by the location of the device in the PCI topology.

- o*N:* indicates that this is an on-board device and the server's firmware provided index number N for the device.
- s*N:* indicates that this device is in PCI hotplug slot N. So ens3 is an Ethernet card in PCI hotplug slot 3.
- p*M*s*N* indicates that this is a PCI device on bus M in slot N. So wlp4s0 is a WLAN card on PCI bus 4 in slot 0. If the card is a multi-function device (possible with an Ethernet card with multiple ports, or devices that have Ethernet plus some other functionality), you may see f*N* added to the device name. So enp0s1f0 is function 0 of the Ethernet card on bus 0 in slot 1.

To identify interfaces `ip link show`

To display IPs: `ip addr show ens3`

To display stats: `ip -s link show ens3`

## Hostname

View

- `hostname`
- `hostnamectl —static`
- `uname -n`
- `nmcli general hostname`
- `/etc/hostname`

Change

- `/etc/hostname → sudo systemctl restart systemd-hostnamed`
- `nmcli general hostname hostname`
- `sudo hostnamectl set-hostname hostname`

- Static Hosts Table: `/etc/hosts`

```bash
192.168.0.110 server10.example.com server10
192.168.0.120 server20.example.com server20
```

- Dynamic Host table to configure nameserver `/etc/resolv.conf`

## IPV4 / IPV6

**Key files:**

- Protocolos: `/etc/protocols`
- Well-known ports: `/etc/services`
- Check ports `ss -lt`
- Each network connection has a configuration file that defines IP assignments and other relevant parameters for it. The networking subsystem reads this file and applies the settings at the time the connection is activated.

    `/etc/sysconfig/network-scripts`

We can create a network connection in two ways

1. Writing a script
2. Using network manager

**Commands**

- `IP`  monitoring, managing interfaces, connections, routing, traffic
- `ifup` Activate a connection
- `ifdown` Deactivate a connection
- `NetworkManager` default interface and connection configuration, administration, and monitoring service used in RHEL 8.

## Network manager

- A daemon that monitors and manages network settings, and saves configuration files in the `/etc/sysconfig/network-scripts`
- `nmcli` is a NetworkManager command line tool that is employed to manage network connections and to control and report network device status. Categories:
  - general
  - networking
  - connection - a collection of settings, only one connection can be active for any one device at a time.
  - device - network interface
  - radio
  - monitor
  - agent

**Device**

```bash
#status of all network devices
nmcli device status

```

**Connection**

```bash
#Show connections
nmcli conn show
nmcli con show -- active

#Adding a network connection with DHCP
nmcli con add con-name eno2 type ethernet ifname eno2

#Adding/delete  connection with IP static
nmcli con add con-name eno2 type ethernet ifname eno2  ip4 192.168.0.5/24 gw4 192.168.0.254
nmcli con add con-name eno2 type ethernet ifname eno2  ip6 2001:db8:0:1::c000:207/64 gw6 2001:db8:0:1::1 ip4 192.0.2.7/24 gw4 192.0.2.1
nmcli con del static-ens3

#Conection up
nmcli con up static-ens3
#deactivate connection
nmcli dev dis ens3

#Modify network connection
nmcli con mod name
nmcli con del static-ens3
nmcli gen permissions
```

### Editing network files

By default configuration created with nmcli are saved on : `/etc/ sysconfig/network-scripts/ifcfg-name`.

If edited we must reload the configuration with `nmcli con reload`

```bash
BOOTPROTO=none #NO DHCP
BOOTPROTO=dhcp
IPADDR0=192.0.2.1
PREFIX0=24
GATEWAY0=192.0.2.254
DNS0=8.8.8.8
PEERDNS=no
```

### Route network traffic same interface they come

This feature can be implemented by using policy-based routing (source routing).The **iproute** package provides the tools (/sbin/ip) to configure this.
1.

El problema viene si tenemos dos interfaces con diferentes GW y queremos asignar dos GW

Tenemos estas interfaces

```bash
ens160 -> 172.28.14.8 ->gw 172.28.1.5
ens192 -> 172.16.29.31 ->gw 172.16.29.254

```

Setup 2 more routing tables with different table ID

```bash
ip route add 172.28.0.0/16 dev ens160 table 1
ip route add default via 172.28.1.5 dev ens160 table 1
ip route add 172.16.29.0/24 dev ens192 table 2
ip route add default via 172.16.29.254 dev ens192 table 2
```

Create rules to forward all the packets entering via a particular NIC to go out the appropriate routing table.

```bash
ip rule add iif ens160 table 1
ip rule add iif ens192 table 2
```

Create rules to forward packets to go out the specific routing table.

```bash
ip rule add from 10.66.1.51 table 1
ip rule add from 10.67.1.51 table 2
```

Para hacer los cambios permanentes es necesario configurar una serie de ficheros:

# References

[https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_nmcli](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configuring_ip_networking_with_nmcli)

[https://access.redhat.com/solutions/288823](https://access.redhat.com/solutions/288823)

[https://access.redhat.com/solutions/19596](https://access.redhat.com/solutions/19596)

[https://access.redhat.com/solutions/1257153](https://access.redhat.com/solutions/1257153)
