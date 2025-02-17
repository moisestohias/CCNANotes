>Note: there are some repeated sections 

show processes cpu !displays the CPU utilization.
ip routing !Enable routing 

### Adding default route to Windows PC:
-route print: print the routing table.
-You can add static route to Windows PC routing table using: route add command
route add <IP> mask <mask> <egress-interface> metric <lower metric if there are other default route>
eg; route add 10.0.10.0 mask 255.255.255.0 10.0.0.1 metric 5

## Lesson 3 - viewing the switch information
>Note: There are five IOS modes: - user EXEC mode, privileged EXEC mode, global configuration mode, setup mode, and ROM Monitor mode. The first three modes are used to view current settings and configure new settings or modify existing settings.

# IOS Version
`show version`: displays informations about the platform series, IOS version, the uptime, the reason of the last reboot, the location of the currently loaded IOS image, RAM size, the installed interfaces, NVRAM size, the exact model number...

>Note: If are planning on performing IOS upgrade, you first need to make sure that you have enought memory, show version outputs free/used memory in bytes, add these two values to figure out how much memory in total that you have.

# Interface Status
+ `show ip int brief`: lists all interfaces, their IP and status.
+ `show interface <Int-ID>`: to show the full info about a port
>Warning: pay attention to the duplex and speed, both of theme are negotiated.
>Note: Note that a duplex mismatch is one of the most common problems with switches to router connections. This issue arises because the speed and duplex settings are negotiated between devices during the connection process, and if the one device is configured manually and the other peer is set to auto, since the safest bet is to go with half-duplex, and this negotiation can fail. In such cases, some of the data will still pass through, but it results in slow performance and packet loss."

>Note: High CRC and hight late collision, is book indication of Duplex mismatch, the side with high CRC errors will be seen on the side with Full-Duplex, the side with high Late collision errors will be seen on the side with Half-Duplex.
+ `show interface <Int-ID> switchport`: Display all switchport info: {administrative,operational} {mode,encapsulation}, 

# MAC Address Table
+ `show mac address-table`: display the MAC address table of the switch.
+ `show mac address-table dynamic`: display just the dynamic MAC addresses.
+ `show mac address-table static`: display just the static MAC addresses.
+ `show mac address-table interfaces "g0/1"`: display just the MAC addresses learned off of this port.
+ `show mac address-table aging-time`: not found in Packet-Tracer.
+ `clear mac address-table dynamic`

>Note: the MAC table (CAM) has dynamic and static addresses, most of the static addresses are used to for Switch features (not available in Packet-Tracer). The first static address is the one used for CISCO discovery protocol CDP, that allows switches learn about neighboring switches.

- The Aging Time:
the switch continually update its MAC table by removing mac address of the devices that are not active, the time for an address to be removed from the table is called the aging time, and it's 5m(300s) by default.
> show mac address-table aging: to display the  aging time
> (config)# mac address-table aging-time "second" : change the global aging-time
> (config-if)# switchport port-security aging time "minute": change the aging-time per port.

- Set static MAC address entry:
# mac address-table static "MAC" vlan "#" interface "Xx/x": set a static mac address entry


# Port Security
## port security is feature that allow us to protect the network by configuring rules and actions that will take place based on those rules.

>Warning: to be able to set the port-security functionality the port should be in STATIC ACCESS MODE. *

> (config-if)# switchport port-security: enable port security on this interface.

#### Specifying the maximum MAC address:
> (config-if)# switchport port-security maximum "max-device" : to set the maximum number of devices allowed on this port.

>Note: there are two ways to set the allowed mac addresses :
- manually > (conf-if)# switchport port-security mac-address "mac"
- sticky > (conf-if)# switchport port-security mac-address sticky
>Note: in case of sticky, the first mac addresses learned of a the port those who will be allowed, btw those permitted mac addresses will live in the running-conf if the switch reboot they will vanish, you need to save the running-configuration to the start-configuration after learning them, unless you want to permit only two devices and you don't care who the are*

## Violation Settings
Beside specifying the maximum number of devices  allowed, you can also configure the violation settings, what the switch will do when a violation occurs (disallowed mac address show up), there are three mode:
+ **protect**: drop traffic of disallowed mac addresses.
+ **restrict**: drop traffic of disallowed mac addresses , increment the security violation counter.
+ **shutdown**: shut down the port, the port goes into ERROR DISABLE STATE and also sends SNMP (Simple Net Management Protocol) message to SNMP server if configured, and to recover from ERROR DISABLE STATE you resolve what caused it, disconnect the disallowed mac address and shut down the interface and bring it back again.
The MAC that caused the violation will be saved in the running config.
>Note: If the port is configured to shutdown, the violation counter will increment after a violation occur, and the port enter a error disable state, if you resolve the issue and bounce the interface the violation counter will reset to zero, so it's not really helpful in shutdown mode.
# show int status err-disable: displays only the err-disabled interfaces and the cause.

>Note: When a violation occur you will get a log that looks something like this:
```
*Nov 25 15:07:12.242: %PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address 0050.7966.6800 on port Ethernet1/0.
```

## Err-disable auto Recovery
You can configure error recovery on the switch to bring the port up automatically again after a violation has occurred.
(config)# errdisable recovery cause psecure-violation
!Brings a secure port out of error-disabled state
(config)# errdisable recovery interval <time-iterval>
!set err-disable recovery attempt interval

> show port-security : display the port-security configuration,
> show port-security interface "int": display the port-security configurations and state for that interface.
> show port-security address: display the allowed MAC address in the entire switch and how they are learned static or sticky.
> show port-security address interface "int": display the allowed MAC address on that interface and how they are learned static or sticky, and other information as well.

>Warning: if you're planning on using port security on port that connect to another switch, you need to account for the switch MAC address as well.


# Hostname and password
> (config)# hostname "BJH": changing the hostname.
> enable password: assigns a password to the privileged mode, but the password will stored in clear text in the running-configuration, so we it's best to use enable secret instead
> enable secret "psswd": displays hashed value of the password.
>Note: the hash value can not be decrypted or reverse engineered.

>Note: if enable password and enable secret are both set, the enable secret is the one that will end up used, in fact the  enable password is very old command and you should never use it.

> service password-encryption : to encrypt the console and the vty and the user name passwords so they are not in clear text.
>Note: the encryption algorithm used is type 7 algorithm which is weak and can be easily decrypted, so it is used only to hide the password from been stored in plain text.

# Setting the Exec timeout
- by default after a period of inactivity the line (console or vty) will timeout and exit, you can set the timeout period or disable it.
> (config-line)# exec-timeout "min" "sec" : 0 0 means don't timeout

## Creating banner
### banner can be set to display a message to the user,
+ banner ? : to see what banner you can set on the device
>Note: MOTD sometimes cannot be displayed when connecting using ssh, login is the most used.
+ banner login "message": put the msg between two character delimiters, you can use any character as a delimiter.

## Specifying the port speed and Duplex
Most Ethernet ports speed and duplex mode can be changed:
(config-if)# speed ? : to change the port speed.
(config-if)# duplex ? :(auto, full, half): to change the port duplex:

MDIX : Media Dependent Interface crosXover. is auto and can't be change.

## CDP
CISCO Discovery Protocol (CDP): Cisco propitiatory protocol used to discover neighboring devices(directly connected)
>Note:CDP must be enabled on an interface for a voice access port to work with Cisco IP phones.
show cdp
show cdp detailed : to display detailed information about the neighboring devices.
+ To turn off CDP globally : # no cdp run
+ To turn CDP globally back on: # cdp run
+ To turn CDP version 2 off so you only use version 1: # no cdp advertise-v2 , (not a good idea)
+ To turn CDP for specific interface, from that interface: (config-if)# no cdp enable, so useful if that specific interface is connected to another autonomous system that's not under our administration.
*CDP is nice option for small network, but isn't that great for large scale network with 100s of devices*

# arp "IP-address" : display the mac address for the specified address if it exist on the arp cash
