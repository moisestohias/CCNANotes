---
marp: true

---

# Router Configuration

---

## Lesson 1: Router Operations

---
<style>
section {
    font-size: 20px;
}
</style>
## Router Interfaces type
+ **Console**: is the port to connect to the router using Rollover cable to access the router's CLI, to configure it.
+ **Auxiliary (Aux)**: is the port that connect to digital modem to access the device remotely (deprecated).
+ **Mini USB**: most modern devices have this port to access the device without Roll cable.
- **Roll (RollOver) Cable**: has RJ-45 jack at one of its ends and DB-9 at the other end.
>Note: Most modern laptop don't have this DB-9 port, so it's common to use USB to Serial adapter.

---

## Connecting the router to other routers:
to connect the router to other routers, we might do that on the LAN using some speed of Ethernet or we can do that on the WAN using a Serial interface.
- **Smart Serial Connector** : is a small form-factor, 26 pin serial connector, used to connect a CISCO router to a serial device CSU/DSU.
- **EHWIC** : Enhanced High-Speed WAN Interface Card.
<!-- ![Smart Serial Connector cable](https://m.media-amazon.com/images/I/71oFvv3l6cL._AC_SX425_.jpg)
![Smart Serial Connector port](https://www.certificationtrainingsolutions.com/wp-content/uploads/2017/10/WIC-2ASb.jpg) -->

<style>
  .side-by-side {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .side-by-side img {
    margin: 0 10px;
  }
</style>

<div class="side-by-side">
  <img src="https://m.media-amazon.com/images/I/71oFvv3l6cL._AC_SX425_.jpg" alt="Smart Serial Connector cable" width="300px" />
  <img src="https://www.certificationtrainingsolutions.com/wp-content/uploads/2017/10/WIC-2ASb.jpg" alt="Smart Serial Connector port" width="300px" />
</div>
---

## Loopback interfaces:
+ Loopback interfaces are virtual interfaces internal to CISCO IOS.
+ Created loopback `(config)#interface loopback "ID"` : 'ID' is an integer.

Once configured, that loopback interface exists inside that router IOS and is not tied to any physical interface. A loopback interface can be assigned an IP address, routing protocols can advertise about the subnet, and you can ping/traceroute to that address. It acts like other physical interfaces in many ways, but once configured, it remains in an up/up state as long as:
+ The router remains up
+ You do not issue a shutdown command on that loopback interface

---

## Terminologies
+ **Packet forwarding**: refers to the process of how the router does its job internally, of forwarding packet to their destination;
+ **Process switching**: the CPU is involved in every packet forwarding decision.
+ **Fast Switching**: the router stores the routes that it routed in Route Cache. so the next packets going to the same destination just use the information stored in the cash.
+ **CEF (CISCO Express Forwarding)**: the CPU proactively populate a cash, which is divided into two parts:
    + **FIB (Forward Information Base)**: stores layer 3 routing information (routes and their next-hop IP address), in a very efficient table structure.
    + **Adjacency table**: stores information about how to construct layer two header for the next hop IP address (egress interface, encapsulation, mac address).

---

## [Data, Control, Management Plane](ciscopress.com/articles/article.asp?p=2272154&seqNum=3)
The work performed by a networking device can be divided into three broad categories:
- The Data Plane: is the forwarding plane, which is responsible for the switching of packets through the router (that is, process switching and CEF switching). In the data plane, there could be features that could affect packet forwarding such as quality of service (QoS) and access control lists (ACLs)
- The Control Plane: is the brain of the device and refers to the configuration and processes (protocols) that control and change the choices made by the data plane. The network engineer can control which interfaces are enabled and disabled, which ports run at which speeds, how Spanning Tree blocks some ports to prevent loops, and so on. The control plane is responsible for maintaining sessions and exchanging protocol information with other router or network devices.

---

>Note: most of the CPU power is spent on the control plain.
- The management plane: is used to manage a device through its connection to the network. Examples of protocols processed in the management plane include SNMP, Telnet, FTP, S-FTP, SSH, etc. These management protocols are used for monitoring and for CLI access.

---

## Lesson 2: Router Hardware

---

### Router's LEDs: similar to the switch's LED.
### Interface Addressing:
Interface Addressing is the same way we address interfaces on the switch (Module/slot/port), but the indexing start from BOTTOM TO TOP, RIGHT TO LEFT. >Note: Module 0: is the motherboard.

# Lesson 3: Logging in
# Connecting To The Router :
>Note: you can't telnet to a switch or router until you configure the vty lines with password.
*maybe because of the login command that is set by default*

---

>Note: When connecting to the Device through USB, that USB connection looks like COM (Communication) port in the terminal emulation software, so it is important to configure the appropriate BAUD rate under CONNECTION ## SERIAL, in the Terminal emulator software, CISCO routers by default has :

| Atribute    | value                   |
|-------------|-------------------------|
|Speed        | 9600                    |
|Data Bits    | 8                       |
|Stop Bits    | 1                       |
|Parity       | None (no parity number) |
|Flow Control | XON/XOFF                |

---

```
# enabling telnet:
(config)#username cisco password cisco
(config-line)#line vty 0 - 15
(config-line)#login local

# enabling ssh:
(config)#username cisco password cisco
(config)#ip domain-name cisco.com   # any domain-name
(config)#crypto key generate rsa modulus "number greater than 756 if you're using SSH v2"
(config)#ip ssh version 2
(config)#line vty 0 - 15
(config-line)#login local
```

---

`show inventory` : display a router's component cards and modules with number of the slot

---

## Cisco CLI features
>Note: the 'Pipeline' character is used to pip the output of one command to another, mainly used for output modifier.
+ `show run | b line` : (b = beging) display run-conf starting from the first occurrence of the word "line".
+ `show run | s line` : (s = section) display all sections of run conf that contain the word "line".
+ `show ip int br | e unassigned` : (e = exclude) display only the interfaces that have an IP address.

---

show version
+ `show version` : display many infos about the IOS, and the device itself.
- Version of the IOS
- Ios Series Number & the exact IOS Number
- The Uptime (for how long this device has been powered on)
- Where the IOS is loaded, normally will be in the Flush, but it could be from the TFTP server, like in case of GNS3
- The platform which is the hardware
- The total available memory, which is important when performing upgrade.

---

+ `show running-configuration`
+ `show ip int brief` summery of the status of the int
+ `show ip int "int-id"` full info about that int

---

# IP Routing table
+ `sh ip route`: displays the routing table
### IP routing table codes
+ `C` : Directly Connected network
+ `L` : Local interface, all local interfaces have 32 mask which means that this is not a net that contain multiple IP.
+ `S` : Static route, the default route is always static (0.0.0.0/0)
+ `O` : OSPF

---

|Source| AD  |
|------|-----|
|C     | 0   |
|S     | 1   |
|EIGRP | 90  |
|OSPF  | 110 |
|RIP   | 120 |

---

if there are multiple routes to the same network pointed by different Route Source, there's a hierarchy of believability or trustworthy called Administrative distance,

---

# Neighboring Devices :
CISCO devices have what's called CDP : CISCO Discovery Protocol, that discovers neighboring CISCO devices that are up and also running CDP (the CDP is not disabled) and some info about those layer 2 adjacent CISCO devices.

>Note: CDP is a layer 2 protocol so we don't even need an IP on the nodes for CDP to work

---

+ `sh cdp neighbors` , display summery of adjacent devices, platform, interface type and ID
+ `sh cdp neighbors detail` : detailed info about adjacent devices.
+ `sh cdp entry "device_name"` , or we can use start to get detailed infos about all devices.

---

### Disabling CDP is useful in case of ISP, or something like that
+ `no cdp run` : disable CDP globally on all interfaces,
+ `(config-in)# no cdp enable` : disable CDP per interface.

---

### assigning a password to the privileged mode, we can use :
+ `enable password` (the password is stored as plain text in the running conf ) it's not recommanded
+ `enable secret` (the password is hashed with MD5 (Message Digest Algo) in the running conf ) recommanded.
>Note: Using enable secret is more resistante to Brute Force Attack

# setting the EXEC TIMEOUT, which can be set per line,
# sh run | s line : to see the EXEC timeout
# (config-line)#exec-timeout "num of minute" & "num of second"


### assigning a password to a line :
+ `(config-line)# password "passwd"`: the passwd displayed in clear text.
+ `service password-encryption` : this's type 7 encryption, which is weak and can be easily cracked.

### assigning an IPv4 address
```
(config-if)# ip address "IP" "mask"
(config-if)# ip address "IP" "mask" secondary : "secondary" means this interface could belong to more than one subnet, generally it's not used.
(config-if)# no  shutdownn : it's always good step even if it's not necessary.
```


### Assigning an IPv6 address :
>Note: firt make sure that IPv6 Unicast Routing is enabled, bc it may not by default.
(config)# ipv6 unicast-routing
`sh ipv6 interfaces brief `
(config-if)# ipv6 enable : this will enable IPv6 protocol on the interface and construct link-local address.
(config-if)# ipv6 address "ipv6" "/mask": in IPv6 we can use the slash notation for the mask.
4x :  ipv6 address 2000::4/64

# creating banner:
```
(config)# banner login $ your login message $
(config)# banner motd $ your Message Of The Day message $
```
>Warning: MOTD sometimes wont show up when using ssh, login is the most used.


# Useful Commands
(config)# no ip domain-lookup : disable DNS lookup when you mistype something and it think it's domain-name.
(config-line)# logging synchronous : sometime when you're typing and console message appears, it intersect your typing and it can be hard to keep track of what's up, this command help by putting everything you typed in a fresh line.
(config-if)# description "interface's description": Anything you put in here will appear in the running configuration right under the interface.
+ `#debug ip icmp`: displays the icmp replies.

>Note: in the router if an interface was brought up using "no shutdown" command and assigned IP address, but no device is connected to that port, or the other device interface is administratively down, the status of that interface, will be UP/DOWN.