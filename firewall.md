# Linux-Firewall

# Concepts

RHEL is shipped with a host-based firewall solution that works by filtering data packets.

Composed by:

- Netfilter: The Linux kernel includes netfilter, a framework for network traffic operations such as packet filtering, network address translation and port translation. By implementing handlers in the kernel that intercept function calls and messages, netfilter allows other kernel modules to interface directly with the kernel's networking stack. Firewall software uses these hooks to register filter rules and packet-modifying functions, allowing every packet going through the network stack to be processed. Any incoming, outgoing, or forwarded network packet can be inspected, modified, dropped, or routed programmatically before reaching user space components or applications.
- Nftables: a new filter and packet classification subsystem that has enhanced portions of netfilter's code, but retaining the netfilter architecture such as networking stack hooks, connection tracking system, and the logging facility. The advantages of the nftables update is faster packet processing, faster ruleset updates, and simultaneous IPv4 and IPv6 processing from the same rules. Another major difference between nftables and the original netfilter are their interfaces. Netfilter is configured through multiple utility frameworks, including iptables, ip6tables, arptables, and ebtables, which are now deprecated. Nftables uses the single nft user-space utility, allowing all protocol management to occur through a single interface, eliminating historical contention caused by diverse front ends and multiple netfilter interfaces.

# Firewalld

`Firewalld` is a dynamic firewall manager, a front end to the nftables framework using the nft command

Firewalld checks the source address for every packet coming into the system.

- If that source address is assigned to a specific zone, the rules for that zone apply.
- If the source address is not assigned to a zone, firewalld associates the packet with the zone for the incoming network interface and the rules for that zone apply.
- If the network interface is not associated with a zone for some reason, then firewalld associates the packet with the default zone

NetworkManager can be used to automatically set the firewall zone for a connection

Configuration:

- Default rules: `/usr/lib/firewalld`
- Custom rules:`/etc/firewalld`

### Zones

Zones define policies based on the trust level of network connections and source IP addresses

- Configuration may include services, ports, and protocols that may be open or closed.
- Rules for each zone are defined and manipulated independent of other zones

By default, all zones permit any incoming traffic which is part of a communication initiated by the system, and all outgoing traffic.

| Zone Name | Default conf. |
| --- | --- |
| trusted | Allow all incoming traffic. |
| home | Reject incoming traffic unless related to outgoing traffic or matching the ssh, mdns, ipp-client, samba-client, or dhcpv6-client pre-defined services. |
| internal | Reject incoming traffic unless related to outgoing traffic or matching the ssh, mdns, ipp-client, samba-client, or dhcpv6-client pre-defined services (same as the home zone to start with). |
| work  | Reject incoming traffic unless related to outgoing traffic or matching the ssh, ipp-client, or dhcpv6-client pre-defined services |
| public | Reject incoming traffic unless related to outgoing traffic or matching the ssh or dhcpv6-client pre-defined services. The default zone for newly added network interfaces. |
| external | Reject incoming traffic unless related to outgoing traffic or matching the ssh pre-defined service. Outgoing IPv4 traffic forwarded through this zone is masqueraded to look like it originated from the IPv4 address of the outgoing network interface |
| dmz | Reject incoming traffic unless related to outgoing traffic or matching the ssh pre-defined service |
| block | Reject all incoming traffic unless related to outgoing traffic |
| drop | Drop all incoming traffic unless related to outgoing traffic (do not even respond with ICMP errors). |

**Service configuration files**

firewalld stores zone rules in XML format at two locations:

1. System-defined rules: /usr/lib/firewalld/zones
2. User-defined rules in the /etc/firewalld/zones directory.

- A system zone configuration file is automatically copied to the /etc/firewalld/zones directory if it is modified with a management tool.
- You can copy the required zone file to the /etc/firewalld/zones directory manually, and make the necessary changes.

### **Pre-defined Services**

We can use services for easier activation and deactivation of specific rules.

firewalld has services pre-configured for various services and stored in different files.

- `firewallcmd --get-services` to list them
- Examples: ssh, samba-client..

## Configuration

3 ways:

1. Edit configuration files
    1. default rules in files located in the /usr/lib/firewalld directory
    2. Custom rules in the /etc/firewalld directory. The default rules files may be copied to the custom rules directory and modified
2. Web Console graphical interface
3. Firewall-cmd command-line

### **Firewall-cmd**

The firewall-cmd command interacts with the firewalld dynamic firewall manager.

Almost all commands will work on the runtime configuration, unless the `--permanent` option is specified.

- If the `--permanent` option is specified, settings must be activated by also running the `firewall-cmd --reload` command
- Option `--zone=zone` to determine what zone applies

| Commands | Explanation |
| --- | --- |
|  |  |
| `--get-default-zone` | Query current zone |
| `-set-default-zone=ZONE` | Set default zone |
| `--get-zones` | List all zones |
| `--get-active-zones` | List actives zones |
| `--add-source=CIDR [--zone=ZONE]` | Route traffic from IP to a zone, |
| `--remove-source=CIDR` |  |
| `--add-interface=INTERFACE [-zone=ZONE]` | Route all traffic from an interface to a zone. |
| `--list-all [--zone=ZONE]` | List of configured interfaces, sources and ports for Zone |
| `--list-all-zones` | Retrieve all information for all zones |
| `--add-service=SERVICE [-zone=ZONE]` // --remove-service | Add or remove service |
| `--add-port=PORT/PROTOCOL [-zone=ZONE]` //--remove-port | Add or remove port |
| `--reload` | Drop the runtime configuration and apply the persistent configuration |
| `--state` | Displays the running status of firewalld |
| `--permanent` | Stores a change persistently. The change on becomes active after a service reload or rest |

# Examples

Set the default zone to dmz, assign all traffic coming from the 192.168.0.0/24 network to the internal zone, and open the network ports for the mysql service on the internal zone

```bash
firewall-cmd --set-default-zone=dmz 
firewall-cmd --permanent --zone=internal --add-source=192.168.0.0/24 
firewall-cmd --permanent --zone=internal --add-service=mysql
firewall-cmd --reload #activate rule
firewall-cmd --list-services #list service in default zone
```
