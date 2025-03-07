A good practice in IOS shell is to name your variable like Pools and ACLs etc, with upper case, to quickly distiguich between commands part and variables.

In the world of TCP/IP, the word host refers to any device with an IP address: your phone, your tablet, a PC, a server, a router, a switch—any device that uses IP to provide a service or just needs an IP address to be managed. The term host includes some less-obvious devices as well: the electronic advertising video screen at the mall, your electrical power meter that uses the same technology as mobile phones to submit your electrical usage information for billing, your new car.
No matter the type of host, any host that uses IPv4 needs four IPv4 settings to work properly:
+ IP address
+ Subnet mask
+ Default routers
+ DNS server IP addresses


# DHCP Dynamic Host Configuration Protocol
Allows for the dynamic assignment of IP addresses to devices on the network, devices (DHCP clients) can request ip address from DHCP server and the server can hand ip address if ther are any left to devices that are asking.
- aside from an ip address and the mask the DHCP server usually responsible for the ip of the default gateway and the ip of the DNS server.

(config-if)# ip address DHCP : prompt the interface of a router to get its IPv4 address form a DHCP server.

### The Ports Used by DHCP:
+ DHCP server: UDP 67
+ DHCP client: UDP 68

DHCP allows both the permanent assignment of host addresses, but more commonly, DHCP assigns a temporary lease of IP addresses. With these leases, the DHCP server can reclaim IP addresses when a device is removed from the network, making better use of the available addresses.

There are four steps to the DHCP process and are usually abbreviated as DORA.
+ **Discover**: Sent by the DHCP client probing for a DHCP server, it lists a number called the client ID in the discover, which includes the host's MAC address
+ **Offer**: Sent by a DHCP server responding to discover probes offering to lease to that client a specific IP address (and inform the client of its other parameters) 
> Note: The source IP of the offer packet is the IP of the DHCP and the destination is 255.255.255.255 (The packet is also encapsulated in an Ethernet broadcast frame) so all hosts in the subnet receive the Offer message, but notice that the message lists another device's DHCP client ID, so they ignore it, and only the target device respond.
+ **Request**: Sent by the DHCP client to ask the server to lease the IPv4 address listed in the Offer message
+ **Acknowledgment**: Sent by the DHCP server to assign the address and to list the mask, default router, and DNS server IP addresses

The problem with DHCP is the request that it sent by the client looking for an DHCP server is a broadcast, and by default routers blocks broadcast traffic so the request will never reach the DHCP server if that sever is not on the same subnet. and it's a common design to use one centralized DHCP sever for many subnet. So we need to configure the router to forward the broadcast traffic to the DHCP server.

DHCP also enables mobility. For example, every time a user moves to a new location with a wireless device the user's device can connect to another wireless LAN, use DHCP to lease a new IP address in that LAN, and begin working on the new network. Without DHCP, the user would have to ask for information about the local network and configure settings manually

- from the router interface of the subnet that the DHCP client lives in we run the ip helper-address command 
(config-if)# ip helper-address "DHCP IP"
- The ip helper-address "DHCP IP" subcommand tells the router to do the following for the messages coming in an interface, from a DHCP client:
1. Watch for incoming DHCP messages, with destination IP address 255.255.255.255.
2. Change that packet's source IP address to the router's incoming interface IP address.
3. Change that packet's destination IP address to the address of the DHCP server (as configured in the ip helper-address command).
4. Route the packet to the DHCP server

This command gets around the “do not route packets sent to 255.255.255.255” rule by changing the destination IP address. Once the destination has been set to match the DHCP server's IP address, the network can route the packet to the server.

* NOTE: This feature, by which a router relays DHCP messages by changing the IP addresses in the packet header, is called DHCP relay and the router that does this is called the relay agent or 'IOS DHCP relay agent' *

- When a router (DHCP relay agent) receives a DHCP message, addressed to one of the router's own IP addresses, the router realizes the packet might be part of the DHCP relay feature. When that happens, the DHCP relay agent needs to change the destination IP address, so that the real DHCP client, which does not have an IP address yet, can receive and process the packet. when DHCP relay receives the DHCP Offer message sent to it's own address. DHCP relay changes the packet's destination to 255.255.255.255 and forwards it out the interface that is connected to subnet that the DHCP client lives in. As a result, all hosts in that LAN (including the original DHCP client) will receive the message .

DHCP uses three allocation modes, 
> Automatic Allocation: sets the DHCP lease time to infinite. As a result, once the server chooses an address from the pool and assigns the IP address to a client, the IP address remains with that same client indefinitely.
> Static Allocation: pre-configures the specific IP address for a client based on the client's MAC address.
> Dynamic Allocation: The DHCP server leases an address for a limited time (usually a number of days), and then the client can ask to renew the lease. If the client does not renew, the server can reclaim the IP address and put it back in the pool of available IP addresses.

Additionally, the DHCP server can be configured to supply some other useful configuration settings. For instance, a server can supply the IP address of a TFTP server.

- Informations DHCP server needs to know:
> Subnet ID and mask: The DHCP server can use this information to know all addresses in the subnet. (The DHCP server knows to not lease the subnet ID or subnet broadcast address.) 
> Reserved (excluded) addresses: The server needs to know which addresses in the subnet to not lease.
> Default router(s): This is the IP address of the router on that subnet. 
> DNS IP address(es): This is a list of DNS server IP addresses

- To verify the relay agent:
> show interfaces vlan x : just under the MTU the helper address will be set if configured, if there were no ip helper-address commands configured on the interface, the text would instead be “Helper address is not set.”

- e.g Configuring a Switch as DHCP Client
A switch can act as a DHCP client to lease its IP address. In most cases, you will want to instead use a static IP address so that the staff can more easily identify the switch's address for remote management. However, just as an example of how a DHCP client can work : aside from specifying the ip address of the vlan statically like we normally would we use:
> ip address DHCP
> no shutdown : you have to bring the VLAN interface up otherwise is administratively down, and if you dont do that the interface will be in the administratively down - down state, because the switch does not attempt DHCP until the VLAN interface reaches an up/up state.

- To verify that DHCP worked and the interface learend an ip address from the DHCP server :
> show interfaces vlan x : just between the bia & MTU if learned will show up in there as internet address
- If you statically configure the IP address, the IP address will always be listed; however, when using DHCP, this line only exists if DHCP succeeded. Also, note that when present, the output does not state whether the address was statically configured or learned with DHCP.

> show DHCP lease : diplays the (temporarily) leased IP address and other parameters, (Note that the switch does not store the DHCP-learned IP configuration in the running-config file.) 

###  Configuring the router to be the DHCP server
```
ip DHCP pool PoolName
(dhcp-config)# network <NET> <MASK> ! the mask can be slash notation
(dhcp-config)# default-router <Gateway-IP>
(dhcp-config)# dns-server <DNS-SERVER-IP>
(dhcp-config)# ip dhcp excluded-address <START-IP> <END-IP>
```

### Configuring a Router as DHCP Client:
you can configure router interfaces to lease an IP address using DHCP rather than manually static IP address, although those cases will be rare. However, configuring a router to lease an address using DHCP makes sense in some cases with a router connected to the Internet; in fact, most every home-based router does just that.
A router with a link to the Internet can learn its IP address and mask with DHCP and also learn the neighboring ISP router's address as the default gateway
The DHCP process supplies a default gateway IP address to router R1, but routers do not normally use a default gateway setting; only hosts use a default gateway setting. However, the router takes advantage of that information by turning that default gateway IP address into the basis for a default route
the Router dynamically adds a default route to its routing table with the default gateway IP address from the DHCP message—which is the ISP router's IP address—as the next-hop address.

- the default route added to routing table as a result of learning a default gateway address of 192.0.2.1 from DHCP. Oddly, IOS displays this route as a static route (destination 0.0.0.0/0), although the route is learned dynamically based on the DHCP-learned default gateway. To recognize this route as a DHCP-learned default route, the administrative distance value used is 254. IOS uses a default administrative distance of 1 for static routes configured with the ip route configuration command but adefault of 254 for default routes added because of DHCP.

### Identifying Host IPv4 Settings:
- Windows :
ipconfig /all: list the address, mask, default gateway, also the DNS server
netstat -rn : lists the host's IP routing table.

- MacOs :
ifconfig : similar to the Windows ipconfig /all, but does not list the default gateway and DNS servers,
networksetup -getinfo Ethernet : list the address, mask, and default gateway
networksetup -getdnsservers Ethernet : list the DNS server
netstat -rn : lists the host's IP routing table. but with several differences in the output than windows

- Linux : often support a large set of commands. However, an older set of commands, referenced together as net-tools, has been deprecated in Linux, to the point that some Linux distributions do not include net-tools. (You can easily add nettools to most Linux distributions.) The net-tools library includes ifconfig and netstat -rn. To replace those tools, Linux uses the iproute library, which includes a set of replacement commands and functions, many performed with the ip command and some parameters.
ifconfig wlan0: MAC and IPv4 addresses, subnet mask, similar to the macOS. However, on Linux, it also shows some interface counters.
ip address : similar output to ifconfig wlan0 .
netstat -rn : like other OSs list the routing table but much abbreviated
ip route : similar output to netstat -rn



## Network Address Translation (NAT)
NAT allows for the translation of private addresses to public addresses.
>Note: NAT and its variations are different from port mapping (aka port forwarding), since port mapping is used for forwarding incoming traffic from the outside destine to certain port, to private local IP address.

### NAT Termonologies: With NAT there is some termonologies that you need to be aware of:
+ Inside Local   : private addresses inside your network
+ Inside Global  : global addresses attached to your router
+ Outside Global : global addresses in the internet
+ Outside Local  : private addresses outside your network (Non-sense really)

### Static NAT 
Static NAT it's One-to-One mapping (private to public addresses) which can be useful in case if you have a Static global IP address and a server in your network that you want it to be reachable from the outside.
>Note: Static NAT is useful in the case where you have an additional public IP address, and a server that you expose all of its port to the outside.

To configure a static NAT, you go to interface configuration for the inside local and type 'ip nat inside'. and for the global local interface and type 'ip nat outside', basically telling the router which one is the the local and which is the global. 
```
ip nat inside ! on the in facing interface
ip nat outside ! on the out facing interface
ip nat inside source static <host-IP> <Public-IP> !On the out facing interface do the static mapping
```
### Viewing config
`show ip nat translation` ! displays the translation table

## Dynamic NAT
Dynamic NAT is also one to one mapping (private to public addresses) but it's dynamic rather than static. So we can tell the router to assign local hosts a public IP from a pool of IP globally routed ip address, assuming that the ISP gave us multiple globally routed IP address to use. Which is not the case with DSL. 
>Note: If the number of public IP addresses are consumed by active connection, any further translation (connection to the outside) will not be possible.

>Note: Dynamic NAT is not really used much, since it only has very limited use cases. The initial intent of a Dynamic NAT was to allow for protocols which create a second, dynamic connection back to the client. The main example of which is the File Transfer Protocol (legacy implementation), or FTP. FTP clients initiate outbound connections to FTP servers over destination port TCP/21. This connection serves as what FTP considers the control channel. Over the control channel, a FTP client makes a request for a file and provides a random port number to the Server. The FTP Server then initiates a second connection back to the client from source port TCP/20, to the destination port provided by the client in the control channel. It is over this second connection that the file is actually transferred – this second connection is what FTP considers the data channel. The issue is the data channel is a connection initiated from an external host on the Internet, destined to a host behind the Router. In a Dynamic PAT (Port AT), which only allows connections initiated from the internal hosts, the data channel connection would be dropped. But with a Dynamic NAT, the inbound data channel connection would be able to pass through the translation and the clients on the Inside server would be able to successfully use FTP to access files on the Internet.

>Note: The use case above describes the classic implementation of FTP known as Active FTP. There is a more modern implantation of FTP known as Passive FTP which does not require FTP clients to sit behind a Dynamic NAT, and instead allows them to sit behind the much more ubiquitous Dynamic PAT.
[More detail found here](practicalnetworking.net/series/nat/dynamic-nat)

- Just as always the case with NAT or PAT we need to tell the router which interfaces are inside and which are outside local.
- Then we need to tell the router which IP addresses are inside local addresses to do that we use ACL Access Control List, which is generally used to to permit or deny traffics, but in this case it's used to tell the router which IP addresses are inside local:
(config)#access-list "ACL-ID" permit "subnet-ID" "wildcard mask"  
## ACL-ID: the ACL id is a number that determines the types of the ACL
Standard: 1-99 : only filter on IP source or destination or both
Extended: 100- : filter on on IP, TCP/UDP source or destination or both
- Then we need to define the pool of public addresses.
(config)#ip nat pool "Pool name" "Start IP" "End IP" netmask "subnet mask"
- Then just like before we need to map the pool of public addresses with the list of private address, to tie the pool with the list of the private ip :
(config)# ip nat inside source list "ACL ID" pool "Pool name"

## PAT Port Address Translation
- With NAT we had one-to-one mapping but with PAT we have many to one. so we can have multiple devices that use the same IP address to go outside the local network, and that is done throuth port mapping, 
- The router uses port number, to map each device SOCKET (IP and Port combination). 

- normally the router does not use ports, but when using PAT, the router will use ports just like end devices, but for the sole purpose of distinguishing between traffic of the defferent devices. 

- just like with NAT we need to tell the router which interfaces are inside and which are outside local.
```
(config-if)#ip nat inside
(config-if)#ip nat outside
```

- and also just like with NAT we need to tell the router which subnets are inside local (Private), for that we use ACL agian. 
(config)#access-list "ACL ID" permit "subnet ID" "wildcard mask" overload
-note: overload prompt the router to translate whole list of addresses into one address, if you dont include "overload" the router will end up assigning the first ip address from the ACL to that interface.

- then we do the mapping 
(config)#ip nat inside source list "ACL ID" interface "int address" netmask "subnet mask"

-to make sure that the translation take a place you can use: 
>sh ip nat translation, or sh ip nat biding

# NTP (Network Timing Protocol): 
NTP is protocol that allows you to synchronize devices on your network to one clock, so you can doevent correlation, when interpreting the log messages for all devices to easily troubleshoot the network and detect where the problem might be coming from. Another aspect of synchronizing networking devices is for security purpose, digital certificates have an expiration date attached to them, so having the time correctly synchronized for all devices is important to check expired certificates.

- NTP is a UDP based protocol uses port 123, and relies on value called the stratum number to determine the believability of the time source. Like atomic clock on the internet they are supper accurate therefore they have stratum number of one and they are believable.
>Note: technically the atomic clock has a stratum number of zero, the server that gets his clock directly from the atomic clock has stratum number of 1.
- we can configure all devices on the network to get their time information from the internet, but for optimum traffic flow we can configure only one device on the network to get its time information from the internet and serve that information to rest of the devices on the network.
- the stratum number will increment by 1 for each device.
# NTP configurations
>Note: For NTP fast convergence you must configure the clock manually to the current time. 
`#clock set "hour:minuts:second 2d=day literal=month 4d=year" `
>Note: the clock should set from the privileged mode, you need to prefix it with "do" in the global configuration.
- set the timezone (UTC, EST, PST) and the offset. >Note: the time zone name can be any word
(config)#clock timezone "TZ" "offset from UTC-TZ in hour"
- NTP server
(config)#ntp master "stratum number" !setting the device as a NTP server
- NTP client
(config)# ntp server "IP of NTP server" : best to use the loopback interface.
(config)#clock timezone "TZ" "offset from TZ in hour"
(config)#clock summer-time EDT recurring: day ligh saving time

>Note: NTP can take several minutes to synchronize between devices*
- show commands to display ntp informations:
+ `show clock`: display the clock
+ `show ntp association`: displays the ntp server and the source.
+ `show ntp status`: displays the ntp synchronisation state and the strutum number 

# ACL Access Control List
- An ACL can contain multiple entry rules.
- Each entry can be permitted or denied cpecific traffics.
- An ACL can be applied inbound or outbound
- The entries are processed top to buttom (whatever rule comes first is going to take precedance, if you permit an IP and then deny its subnet that IP will still be permited, but if it was the other way around (deny subnet then permit the IP) that wont work the IP will be denied even with the explit instruction to permit it.)
- there is an implicit "DENY ANY" at the buttom of the an ACL 

# Standard ACLs: can be used to permit or deny traffics, base only on the source IP.
>Note: "ACL ID" is a number in the range of 1 - 99 in the case of standard ACL, and it is normally reffered to as just number, it is just me that reffere to it that way.*
(config)# access-list "ACL ID" permit "subnet id" "wildcard mask": permit entire subnet
(config)# access-list "ACL ID" permit host "IP of the host": permit single IP address
(config)# access-list "ACL ID" permit any: permit all devices
* Note : under and standard ACL ther is implicit "DENY ANY" associated with it.*
- An ACL by itself doesn't do anything, it must be applied to a specific interface that is is connected to the host's subnet.
(config-if)# ip access-group "ACL ID" in: in = incomming traffics

> show access-list: display the ACLs on the router

# Extended Numbered ACLs
- Extended ACL can be used to permit or deny traffics, base on both Source or Distination address or the Protocol. the Extended ACLs has number in the range of 100 - 199
(config)# access-list "ACL ID" permit "protocol" host "host IP" eq "protocol name or port number"
- applying Extended ACL to an interface:
(config-if)# ip access-group "ACL ID" in: in = incomming traffics

# Named ACLs
Allows you to use names for both standard and extended ACLs.
```
(config)# ip access-list standard "ACL NAME": Standard Named ACL
(config)# ip access-list extended "ACL NAME": Extended Named ACL
(config-ext-nacl)# permit ip host "HOST IP" host "DESTINATION IP"
(config-ext-nacl)# permit tcp host "HOST IP" host "DESTINATION IP" eq "protocol port number or protocol name"
```
> [!Note] you can have multiple premit statement on the same named ACL

- applying Extended ACL to an interface: `(config-if)# ip access-group "ACL NAME" in`
- Specifying the ACL sequence number manually: 
```
(config)# ip access-list extended "ACL NAME"
(config-ext-nacl)# "SEQUENCE NUMBER" permit or deny..
```