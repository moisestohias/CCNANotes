# VLAN

[you should merge this file](VLANsInfoMege.md)

>Note: when you create VLAN, you are creating a STP instance (assuming PVST/RSTP), and CAM table instance.

>Warning: If you have an end-to-end switched domain, for a packet to go from one device to another device, on the same VLAN, all switches in the transit path must know and trunk that packet's VLAN. For instance if you have a VTP transparent mode (must be in the same domain or NULL) or VTP off mode switch in the transit path that is not learning VTP ads, so it doesn't know about that VLAN, but the ends switches know about that VLAN through VTP, so what's end up happening the end switches knows about the VLANs through VTP but the transparent/off switch doesn't, it doesn't even trunk it.
>Note: the switch don't trunk (thus don't run STP) VLAN, that he doesn't know.
>Warning: All transit siwtches must know about all VLANs in the domain, even if they don't have any port assigned to them, otherwise STP won't work correctly.


VLAN: A technique that lets you have multiple logical LANs operating on the same physical equipment.

>Note: when you disable VTP, the VLAN configuration will be stored in the running-config, and the vlan.dat file won't be used for synchronization. You'll need to manually configure VLANs if needed since there won't be any automatic propagation of VLAN information within the VTP domain.
>Note: if you work with a switch in "Transparent" mode or a switch that doesn't participate in VTP (`vtp mode off`), the VLAN configuration commands should appear in the running-config. However, the switch won't automatically synchronize these VLAN changes with other switches using VTP because it's not participating in VTP upda

### Collision Domains VS Broadcast Domains
- Collision Domain: is a network segment where only one device can talk at a time.
- Ethernet Broadcast Domains: Take any Ethernet LAN, and pick any device. send an Ethernet broadcast.  An Ethernet broadcast domain is the set of devices to which that broadcast is delivered.
>Note: By default, switches break up collision domains and routers break up broadcast domains.

### Ethernet Switches and Collision Domains
- switches perform the same basic core functions as bridges but at much faster speeds, and with many enhanced features. Like bridges, switches segment a LAN into separate collision domains, each with its own capacity. And if no link is connected to a hub, every single link in the switch is considered its own collision domain.

## VLAN definition
A VLAN is a logical grouping of the porst into subnets. When you create VLANs, you’re given the ability to create smaller broadcast domains within a layer 2 switched internetwork by assigning different ports on the switch to service different subnetworks. A VLAN is treated like its own subnet (broadcast domain), meaning that frames broadcast onto the network are only switched between the ports logically grouped within the same VLAN. Within the typical layer 2 network, all users can see all devices by default.  And you can’t stop devices from broadcasting, plus you can’t stop users from trying to respond to broadcasts.

>Note: if you configure VLANs on plain layer 2 switch, devices on different VLANs to be able to talk with each other they need a router, Weird Right!!

VLAN can solve many of the problems associated with layer 2 switching.

## VLAN Creation and deletion
+ `show VLAN brief`: lists known VLANs, and port mapping
+ `show vlan id <id>`: lists VLAN's port mapping

+ `VLAN <word>`: create a new VLAN, word is a number from 1 to 1001, !Only switches configured as "VTP off" are able to create VLANs in the extended range 1006-4094.
+ `name <VLAN-Name>`: to name the newly created VLAN

>Note: by default, all the Ethernet ports of the switch are part of the first VLAN vlan1 which is the native VLAN (concept discussed below), there are other built-in native VLANs (that you will never use like token-ring, FDDI (Fiber Distributed Data Interface)) each switch can have up to 1001 VLANs, and the other built-in from 1002 > 1005.

## Deleting VLANs
+ `no vlan <VLAN-ID>` ! delete a VLAN.
>Note: By default (if you didn't disable VTP) when we create a VLAN, that configuration is stored on a special separate file "vlan.dat" on the switch's FLASH memory, which is the VLAN database, we can display that file by typing: show flash > the name of the file is vlan.dat *

>Warning: before removing any VLAN, you need to reassign all of its ports to another VLAN, if you don't do that, the ports that was part of that VLAN won't be part of any VLAN, as a result you can't uses them until you reassign those ports manually to an existing VLAN.

>Note: The VLAN information exist in file called vlan.dat, which lives on the flash. So if you are planning on recovering the factory default state of the switch, by deleting your configuration, you need to get rid of all the vlan.dat file also to get rid of the VLAN info completely. We can do that on the conf mode:
> delete flash:/vlan.dat : this will erase only vlan.dat from flash .

### Restoring The Factory Default Configuration Of The Switch
(config)#write erase: to erase everything in the NVRAM (the startup-configuration).
>Note: the 'write erase' will only erase the NVRAM content, but not the VLAN database 'vlan.dat' which lives in a totally different space.
> write ? or wr? : to see all the commands available with write.

### Erasing the vlan data base
(config)#delete flash:/vlan.dat : this will erase only vlan.dat from flash .
or:
`#delete vlan.dat`
`#delete filename [vlan.dat]?vlan.dat`
`#delete flash:vlan.dat[confirm] y`
>Note: in IOU it's `#delete unix:/vlan.dat?`
>Note: If you delete vlan.dat, It again could learn the delted information from the vtp database floating in the network if its still connected to the switches via trunk links running VTP, this is due to the VTP advertisements traversing the trunk from the existing, so you first must either disable VTP or the trunk links.
>Note: it's always a good idea to Reload the switch after deleting configuration.

## Assigning ports to a VLAN
When we create a new VLAN we have to manually assign the switch's ports that belong to that VLAN.
To assign a port to a VLAN there are couple ways:
+ 1 Assign an individual port to a specific VLAN: you should be in interface configuration  for a specific Ethernet interface: interface fastEthernet0/0 > switch the state and the VLAN of the port : switchport access vlan 10
- 'access' is the mode of the operation of that specific port as suppose to trunk port that we will discuss later

+ 2 Assign range of ports to a specific VLAN
conf mode > select a range of Ethernet interfaces: `int range fa0/5-10` ! Any modification will affect all the selected interfaces
>Note: you can specify multiple ranges:  `interface range fa0/9-16,fa1/0-12`
>Warning: you should not include any space in the command above of range
switch the state and the vlan of the ports: switchport access vlan 10
access is the mode of the operation of that specific port as suppose to trunk port that we will discuss later

e.g:
```
Switch(config)#vlan 2
Switch(config-vlan)#name ACCT
Switch(config-vlan)#exit
Switch(config)#vlan 3
Switch(config-vlan)#name ENGINEERING

Switch(config)#interface range fa0/9-16
Switch(config-if-range)#switchport access vlan 2
Switch(config)#interface range fa0/17-24
Switch(config-if-range)#switchport access vlan 3
```

>Note: If you assing ports to a VLAN that has not been created yet, the VLAN will automatically created, with the default naming template vlan<id>
```
SW1(config-if-range)#switchport access vlan 3
% Access VLAN does not exist. Creating vlan 3
```

# Lesson 6 Configuring Trunks
### Trunk theory
Practically, **Trunks allow VLANs to span across multiple switches.**
*TRUNK* is a conduit that carries traffic for multiple VLANs by tagging each frame with the appropriate VLAN identifier. In another word, VLAN taggin is how we tell the frame apart on that trunk depends on the trunking technology.
>Note: We mainly use IEEE standard 802.1Q , there's also ISL(Inter Switch Link) which is CISCO proprietary that's not used anymore in the modern CISCO switches, most modern Cisco switches (even older ones 2960s) support only 802.1Q
>Note: the *trunk encapsulation means*: what protocol is going to be used to tag the different VLANs.


## Making sure that devices are using DOT1Q
When configuring a switch it's important to change the trunk encapsulation mode from `negotiate` to trunk, because in older IOSs it might end up negotiating ISL, If both switches support both protocols, they use ISL, but this is still relevent even for modern IOS, since you can configure a port to be trunk untill you specify the encapsulation manually to dot1q
HQ(config-if)#switchport trunk encapsulation dot1q: changes the encapsulation mode to dot1q
>Note: in Packet Tracer only the multi layer switches namely the 3650 and 3560 have this command.

- in the 802.1Q all the frames passing the trunk are tagged (4Byte) except frames that belong to the native VLAN, so each VLAN gets a special # (tag) when the frames get to the other far end switch trunk port, it is going to be able to look at these Ethernet frames and tell which frame belongs to which VLAN.
So both switches have to agree on the Native VLAN  otherwise we might run into problems.

```
                                |-------  4  Bytes  --------|
+-------------------------------------------------------------------------------+
|Preamble|SFD|Destination|Source|Tag Prtcl ID|Tag Control ID|Ether-Type|Data|FCS|
+-------------------------------------------------------------------------------+
```

Tag Prtcl ID (aka Type) 2 Bytes: is the VLAN Protocol ID (0x00008100)

Tag Control ID (2 Bytes) consists of:
3  bits -> QoS
1  bit  -> Canonical Format Identifier (CFI) Flag: Usually 0 meaning canonical format
12 bits -> VLAN ID

>Note: that the maximum user data length is still 1500, so VLAN packets will have a maximum of 1518 bytes (which is 4 bytes longer than usual Ethernet packets) - Excluding the FCS, because this is a Wireshark packe.

>Note: in Wireshark the Tag Prtcl ID (VLAN Type) field is not considered as part of the VLAN tag, but the ether-type is, so in WireShark view the frame looks like: (This is only specific to Wireshark)
```
                                             |-------  4  Bytes  ------|
┌────────┬───┬───────────┬──────┬────────────┬──────────────┬──────────┬────┬───┐
│Preamble|SFD|Destination|Source|Tag Prtcl ID|Tag Control ID|Ether-Type|Data|FCS│
└────────┴───┴───────────┴──────┴────────────┴──────────────┴──────────┴────┴───┘
```

### Ethernet port modes
There are three types of operation for each Ethernet port,
+ **access**: force a port to operate as an access port
+ **trunk**: force a port to operate as a trunk port
+ **dynamic desirable**: initiates the negotiation of a trunk
+ **dynamic auto**: passively wait for the remotes switch to initiate the negotiation of a trunk.

>Note: the switchports by defualts are 'dynamic auto'

|S1                 | S2               | Trunk                |
|-------------------|------------------|----------------------|
|access             |access            | False                |
|trunk              |dynamic desirable | True                 |
|trunk              |dynamic auto      | True                 |
|trunk              |Trunk             | True                 |
|dynamic desirable  |dynamic desirable | True                 |
|dynamic desirable  |dynamic auto      | True                 |
|dynamic auto       |dynamic auto      | False                |
|access             |trunk             | Link goes down by STP|

>Warning: if one side is configured manually to trunk and the other side manually to access, the link will be disabled by STP.

## Dynamic Trunking Protocol (DTP)
Protocol used to negotiate the formation of trunk between switches.
If one or the two switches are configured to negotiate, DTP frame is going to be sent, which allows two switches to negotiate the forming of a Trunk, and if the far-end switch's port is not administratively configured as access trunk will be formed.
>Note: If you are planning on using DTP, you must configured the encapsulation to static dot1q, especially when working in a V-LAB or with old models that support ISL, using the command: "switchport trunk encapsulation dot1q"

>Warning: Even though that DTP can seem cool, but in most design, you don't want automatic un-predictable behaviour, that's Cisco recommends disabling trunk negotiation on most ports for better security.

### Disabling DTP negotiation: to methods
>Note: "switchport mode access" disable DTP negotiation on that port.
Trunk ports are still able negotiation with dynamic desirable, to disable DTP negotiation even on trunk ports use: "switchport nonegotiate" >Note: this command is only available in VIOS-L2.

>Note: DTP frames are sent every 30 seconds trying to form Trunk, If the port is set to Trunk or Dynamic Desirable and the trunk can't be formed -due to VTP domain mismatch- it just keeps trying every 30s. That is why you get those ugly console logs if the trunk can't be formed, in case of VTP domain mismatch.
"#01:30:05 %DTP-5-DOMAINMISMATCH: Unable to perform trunk negotiation on port Gig0/2 because of VTP domain mismatch."

>Note: If the other far-end didn't respond to the DTP frames,(maybe is configured as no negotiate, or it's not the same brand or does not use .1Q standard) the trunk will not be formed and the port will operate as an access port.

>Warning 🪖: for a trunk to be formed between to switches they have to be in same VTP Domain -and if a password is configured a password need to match- if two adjacent switches belong to different VTP domain Trunk link won't be formed and you will get the error msg every 30 s.

"00:17:59 %DTP-5-DOMAINMISMATCH: Unable to perform trunk negotiation on port Gig0/1 because of VTP domain mismatch".

>Note: VTP is discussed below.

## Creating a trunk
Displaying port status
+ `show int fas0/1 switchport`: Displays the administrative, operational modes (access, trunk...), the encapsulation (negotiate, no negotiate)
e.g
- SW1# show interfaces gigabit 0/1 switchport
Name: Gi0/1
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native < reference 802.1Q native VLAN.

- SW1# show interfaces gigabit 0/1 switchport
Name: Gi0/1
Switchport: Enabled
Administrative Mode: dynamic desirable
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q

---

### Show Trunks
+ `show interface trunk`: displays trunks on the switch, if there's any trunk configured and active it'll display it, if not nothing will be printed
+ to change the encapsulation mode to dot1Q instead of negotiate: from the port we want to make as trunk: int# switchport trunk encapsulation dot1q
+ to change the port mode :
(config-if)#switchport mode? > (access ,trunk ,dynamic auto ,dynamic desirable)
if you want the port to generate DTP frames to negotiate the forming of the trunk you need to select: dynamic desirable.

>Warning: If one side of the link is configured as trunk and the other configured as access, the trunk will not be formed and the link will be disabled by STP:
%SPANTREE-2-RECV_PVID_ERR: Received 802.1Q BPDU on non trunk GigabitEthernet0/1 VLAN1.
%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking GigabitEthernet0/1 on VLAN0001. Inconsistent port type.


## Specifying the native VlAN
To change the native Vlan: from the trunk port
> (config-if)# swtichport trunk native vlan 'the VLAN number'
>Note: if we change the native VLAN on one of the switches we have to change in the other peer as well, otherwise is not going to function properly, the technical name for Native VLAN mismatch is called VLAN Hopping.

# Inconsistent Native VLAN on Trunk link:
>Warning: if two sides (trunk port of each switch) don't agree on the native VLAN, the link will be disabled by STP. You will see console log indicating that.

1. The first log that you get after you have Native VLAN mismatch on a trunk is the STP blocking.


```
%SPANTREE-2-RECV_PVID_ERR: Received BPDU with inconsistent peer vlan id 2 on GigabitEthernet0/2 VLAN1.
%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking GigabitEthernet0/2 on VLAN0001. Inconsistent local vlan.
```
Or
```
%SPANTREE-2-RECV_PVID_ERR: Received 802.1Q BPDU with inconsistent peer vlan id "x" on "Interfacex/x" VLANy.
%SPANTREE-2-RECV_PVID_ERR: blocking "Interfacex/x", on vlanx Inconsistent peer vlan.
%SPANTREE-2-RECV_PVID_ERR: blocking "Interfacex/x", on vlany Inconsistent peer vlan.
```

2. As long a Native VLAN mismatch is present you will get CDP log.
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on "GigabitEthernet0/1" (2), with "S1" "GigabitEthernet0/2" (1).

3. After MATCHING the Native VLAN again, STP unblock the link, and you will get console logs, indicating that the interfaces has recovered
S1(config-if)#%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking GigabitEthernet0/2 on VLAN0002. Port consistency restored.

## Implementing ports connecting to IP phone:
>Note: CDP must be enabled on an interface for a voice access port to work with Cisco IP phones.
Access Ports connecting to IP phone which then connect to end device, must be configured to support VLAN tagging, using the `switchport voice vlan <VLAN-ID>` interface command.
Or you can use the old method of doing it (Not recommended), which is to configure a trunk allowing only two VLANs (voice and data) and setting the data VLAN to be the native on that port (aka mini-trunk)
```
switchport mode trunk
switchport trunk allowed vlan 10,20
switchport trunk native vlan 20
switchport nonegotiote !Disable negotiation with 'Dynamic Desirable' ports.
```

## Limiting VlANs on a trunk
We can manually set which VLANs are allowed to cross that trunk and which is going to get blocked.
>Note: by default all newly created VLANs are allowed to pass the trunk *

(config-if)# switchport trunk allowed VLAN ?
- {VLANs} : specify the VLANs you want to allow.
-all: allow all VLANs
-non: block all VLANs
-add "VLANs": new VLAN
-except "VLANs": allow all VLANs Except the following
-remove "VLANs": block one of the allowed VLAN

### VTP (VLAN Trunking Protocol) #################################
"vtp mode off" disable VTP globally.
>Warning: Any switch that uses Server or CLient VTP mode won't list VLANs in the running/startup config.


VTP is a protocol that FLOW OVER A TRUNK and it carries VLAN information, so when a new VLAN is created on one of the switches a VTP frame is going to go over the trunk, to the other switches to informs them that a brand new VLAN is created and they add it to their database. So VTP allows us to create one VLANs on one switch and has that change propagated to the other switches.

- since the inception of VTP and for a long time, most switches did not allow VTP to be disabled completely; on those switches, to effectively disable VTP, the engineer would set the switch to use VTP transparent mode (with the VTP mode transparent global command). Some switches now have an option to disable VTP completely with the "vtp mode off" global command.

>Note: VTP advertises VLAN info only between switches that are part of the same defined domain (not the default NULL domain)

>Note: that both transparent and off modes prevent VTP from 'learning' and advertising about VLAN configuration. Additionally, switches using transparent or off modes, list the VLAN configuration commands in the running-config file. so if VTP wasn't disabled don't expect to see VLAN info in the running-config *

+ VTP Pros :
  - Eliminates a lot of Administrative Burk, if you create a VLAN on one switch, that information can propagate to the other switch through Advertisement so the rest of the switches in the network will know about this VLAN.
  - Eliminates a lot of unnecessary traffics (VTP Pruning).

+ VTP Cons :
  - Many people suggest to not use VTP since in working environment there are different brand of switches, so using VTP won't be feasible, on the contrary VTP can introduce a lot of issues and headache.
  - because the VTP WIPE-OUT many Net-Engineers prefer not to use VTP, and argue that the unnecessary traffic isn't that large to affect the network performance.

### VTP modes
There are three modes of VTP operation: **Server**, **Client** and **Transparent**.
+ **Server and client** are almost the same except that you can't modify VLANs on client, but both receive, update, forward and generate VTP adds.
>Note: The server switches can configure VLANs in the standard range only (1–1005).
+ Transparent (version 2): it's what it sounds like, forward received VTP adds only if it's part of the same or the NULL domain, but doesn't update nor generate VTP adds.
>Note: Transparent VTP version 1, DOES NOT forward received VTP adds, even if it's configured as part of the same domain,
+ Off: it's what it sounds like, forward received VTP adds, but doesn't update nor generate VTP adds.

>Note: that both transparent and off modes prevent VTP from learning and advertising about VLAN configuration. Those modes allow a switch to configure all VLANs, including standard and extended-range VLANs. Additionally, switches using transparent or off modes list the vlan configuration commands in the running-config file.

>Note: A Catalyst switch can participate in only one VTP domains
>Note: All switches by default don't belong to any VTP Domain Name, you have to manually assign VTP domain name to at leas one of the switches - the domain will be propagated by VTP through trunks - and VTP can take place and advertise about VLANs.
>Warning: creating or modifying the domain name forces the ConfRevNum to jump back to zero.
>Note: all switches by default are set to Servers mode

## VTP Pruning

If an access switch doesn't have access ports for certain VLANs, VTP will tell the other switches to not send traffic of that VLAN to this switch so the traffic will be pruned.
>Note: VTP pruning can only performed on trunk between two switches running as Server or Client.
>Note: Only VLAN in the standard range 2-1001 can be pruned.
You can tell the switch which VLAN it can purne on per trunk basis, using "switchport trunk pruning vlan <VLAN-ID>"
To verify VLANs that are pruned: "show interface pruning", the first list is the pruned VLANs, the second list is the VLAN that the local switch is asking for from its neighbor.

## VTP Pruning corener cases
>Note: On trunk ports connecting to Hypervisors or devices that don't implement VTP or running transparent mode, you must configured the allowed VLAN manually otherwise VTP device will end up asking for ALL VLANs it knows about, which will partially cancel out the benefits of VTP pruning, because it doesn't know what the other end have it term VLANs.

>Warning: if you are planning on using VTP pruning in your network, don't have switches running in the transparent mode. Especially on transit switches, because what will end up happening is the switch will forward the VTP traffic request of other switches, but it won't ask for its VLANs.


## Configuration Revision Number:
Every time a database change is made there is a number called a 'Configuration Revision Number' that is associated with the switches VLAN database - this includes Creating, Naming , Renaming, Deleting VLAN.- and that number get incremented by one for every change made. When a change occurs in one of the switches that has not being configured to be transparent it generates VTP Advertisements, when the other switches receive that ADs it compares it to its local ConfRevNum, and if it's greater, then it's going to update its local database based on the received Infos.
-When all switches in the network agree on the same common VLAN database (they all have the same configuration revision number) synchronization is complete.

>Note: There three versions of VTP 1,2,3, the big difference between version one and two is that in version 1, if the transparent mode switch is not in the same domain, it will block VTP messages. We primarily use Version 2, since the transparent switch wont block VTP adds even if it belong different VTP domain.
Version 3 is supported on the newer CISCO switches, it sends extended VLAN (VLANs in a range of 1006 to 4094), it's able to advertise private VLAN.

## VTP Domain
In order for VTP to work the switches that are going to be exchanging VTP information they need to belong to the same domain, if we have a password set up the password need to match.
>Warning: Domain name and password are case sensetive.
### VTP domain mismatch
>Note: if two adjacent switches belong to different VTP domain Trunk link won't be formed and you will get Mismatch Error console Logs every 30 s:

"00:17:59 %DTP-5-DOMAINMISMATCH: Unable to perform trunk negotiation on port Gig0/1 because of VTP domain mismatch".

## VTP Updates:
-VTP sends VTP message every 5 minutes, which contain just basic Info like VTP domain name, the ConfRevNum, and some other data, in another word it won't create a big impact on the network.
>Note: VTP only goes over a trunk line because is only destined to switches.
>Note: When you make a VLAN modification on a server switch, a VTP advertisement will be send immediately.

>Note: the default VTP version is v2

+ `(config)#vtp version x`: Change the VTP version used
+ `(config)#vtp mode <client, server, transparent>`: Change the VTP mode of the switch
+ `(config)#vtp domain <domain name>`: Create (or change) the VTP domain
+ `(config)#vtp password <passwrd>`: Set a password to the domain
+ `(config)#vtp pruning`: Enable VTP pruning so unnecessary trafic will not be sent across the trunk


>Warning: If you don't create any VTP domain, all switches will belong to their own independent default NULL VTP domain - no domain - and won't exchange VTP message (VLAN database). In order to make all the switch part of the same domain you have to manually assign at least one switch a domain name (assuming the other switches have no VTP domain name and therefore will join the created domain, otherwise you have to manually assign all switches the same domain). The moment you match the VTP domain on server or client switches, the VTP database will synchronize base on the switch with the higher ConfRevNum

### Supper Important.
**if we want to add a new switch to the network - Assuming that it has no VTP domain name, or has one that match the existing one-, this switch may have ConfRevNum higher than the one in the domain and it's not set to Transparent mode (even VTP client mode will originate VTP messages), and that will cause the other switches to update its VLAN database, and this known as VTP WIPE-OUT.
which in most cases, we may not want that because that will get rid of the VLAN configuration of the domain, and replace with the configuration of the newly added switch that has a higher revision number, to prevent that we can set the newly added switch to transparent mode and that will cause the ConfRevNum to become zero, then we can set it back to a client or server and the ConfRevNumb will remain zero if no further change is made, before adding it .**

>Note: When a new switch that has no VTP domain name, create Trunk with one that has VTP domain name, the new switch will join the domain, and if it has ConfRevNum higher than the one in the domain, it's VLAN database will take over the domain and obliterate the existing VLAN database. That's what makes adding new switch to the domain dangerous.


>Note: when you set the VPT mode to transparent, it does not only prevent the switch from learning and advertising about VLANs, it also prevent it from storing the VlANs information it know about in the VLAN database file 'vlan.dat'.

## Adding New Switch To The Production Network:
>Note: by default the all of the switch ports are set administratively to dynamic auto, so they passively wait for DTP messages negotiating the formation of trunk, if no DTP messages are received the port will operate as access port.
So when you connect fresh brand new switch to already working environment, that has VTP domain name and one or more switch operating as VTP Server (ConfRevNum greater than 0), if the port of the pre-switch is configured as trunk or dynamic desirable DTP messages will be originated to negotiate the formation of trunk link, and because the fresh switch ports are by default set to dynamic auto the trunk will be formed, and the VTP protocol will communicate the VTP domain name, and because the fresh switch has no VTP domain name, will automatically join the VTP domain (assuming the VTP domain has no password), and eventually learn about the VLANs in the domain.
- Adding New Switch To The Production Network: Configuration:
```
SW(config)#vtp version 2
SW(config)#vtp domain "Domain Name"
SW(config)#vtp mode transparent : set the ConfRevNum to 0.
SW(config)#vtp mode server : back to default VTP mode, so the switch learn about VLANs.
SW(config)#delete flash:vlan.dat : extra step of cautiousness
```

>Note: BY DEFAULT the switch does not belong to any VTP domain, so if you create and modify VLANs on that switch while it's in that state (Not part of VTP domain), for every change the ConfRevNum is incremented by one, so the ConfRevNum is greater than 0, then you create a VTP domain, the Configuration Revision Number changes to 0 automatically, so it wont generate any VTP information. At least that is the case in Packet Tracer, 2960 switch.

>Warning: Even though, you can have pairs of Trunk ports, agree on a different Native VALN other than the main Native VLAN used in the rest of the Domain, It is impractical neither useful, since all Native VLANs traffic will end up mixed up, as one broadcast domain and you no longer have the concept of VLANs isolation. It's rather confusing.

So all in all, all switch in the VTP domain should all use the same Native VLAN.

# VTP Version 3:
>Note: any switch that support VTPv3, allows for disabling VTP globally (VTP mode off).
### Security feature:
VTP v3 adds a counter measure security feature to protect against the problem of VTP WIPE-OUT, by introducing the concept of the Primary and secondary server. This is an extra layer of server role on top of the VTP modes, by default all devices are secondary servers, only one device on the network is allowed to be the primary, all switches list his MAC as being the primary, and obey hist ads.

## VTP v3 Advertisements
The support for the extended range VLANs, which can create and advertise VLANs in the extended range.
Can create and advertise Private-VLAN
Can advertise MST configuration.

## VTP v3 Password:
VTP as I've mentioned before version 3 adds an extra layer of feature to protection against the problem of VTP WIPE-OUT, by introducing the Primary/secondary server role, and you can assign a password to the domain just like before. Now, you can enforce the switch to prompt the administrator to enter the domain's password when upgrading a switch to be the Primary for the domain, even if the switch is already been authenticated as part of the domain, by adding the "hidden" keyword.
vtp password <pass> hidden !this will store the password in the NVRAM:vlan.dat.
>Note: if you use the "hidden" keyword, "show vtp password" won't show you the password.

## Upgrading a switch to be the primary server of the domain
To upgrade a switch to be the primary server, IT MUST BE RUNNING AS A VTP SERVER MODE, and the configuration is carried out in the user exec mode, because this configuration is not written into either the run nor the startup config, it just take effect in place once the switch is rebooted, the primary role is lost.
vtp primary-server [vlan|mst] [force]
+ `-vlan`: set the switch to be primary server for the VLAN VTP domain advertisement.
+ `-mst` : set the switch to be primary server for the MST configuration advertisement.
>Note: is an empty template for new enhancement that they may add later.
If you append "force" this will take over the primary server role in the domain, if there is an existing Primary server, it must submit. Otherwise it will fail and say there are already a primary server in the domain.
>Note: separating vlan from mst, provide flexibility sine a switch can be the primary VTP server for the VLAN database, but a client for MST.

## Why transporting MST configuration over VTP:
The reason why VTP adds the support for carrying and synchronizing MST configuration, is for solving the problem of MST configuration mismatch (creating multiple region), if you recall from MSTP section that one of the design requirement that we have is, for a switches to be part of the same region the VLAN to instance mapping needs to match along side the region name and revision. For instance if you don't have a VTP version 3, and you add an instance, the switch will be in its own region, till you match the MST configuration in the other boxes.

Can Remote-Span VLAN advertised via VTP? Yes

>Note: To fix the problem of pruning requests are not replied to, you can use the standard solution that works for all VTP version, by using the allowed vlan on the trunk. With VTP version 3 however you can turn VTP off per port "no vtp run", so VTP pruning request are not sent.

## VTP verification:
show vtp status
!Displays the VTP configuration on the switch, and the IP of the last device updated the VLAN database.
show vtp password
>Note: when verifying VTP make sure the MD5 digest matches.


### The benefits of VLANs
+ VLANs increase the number of broadcast domains while decreasing their size.
+ VLANs greatly enhance network security if implemented correctly.
+ Network adds, moves and changes are achieved with ease by just configuring a port into the appropriate VLAN.
+ A group of users that need an unusually high level of security can be put into its own VLAN so that users outside of that VLAN can’t communicate with the group’s users.

---

If you delete a VLAN that has ports assigned to it, even though the ports won't show up in the show vlan, but the configuration of "access vlan <deleted-VLAN-ID>" in the running config, won't be deleted you just need to re-itrduce the VLAN.

## VTP Takeaway
+ VTP advertisements only propagate between devices on a defined VTP domain.
+ If you only create VLANs without specifying a VTP domain name, the VTP wont advertise about these VLANs.
+ Every time you define or change the VTP domain name, the ConfRegNum will jump back to zero. So if you change the domain name after creating your VLANs, the ConfRegNum will go back to zero and if an another switch has a ConfRegNum other than zero and don't belong to any domain, it will join the domain via VTP, and its VLAN database will take over the domain.

### Equal ConfRevNum (ConfRevNum greater than 0):
>Note: in case of tow switches have Configuration Revision Number equal to each other AND GRETER THAN 0, but they both don't belong any VTP domain,
when you create a VTP domain name in any of them, they will compare their ConfRevNum to find out it is the same, in this case they look at the Last Modified Date of the VLAN database
or in another word which database was last modified, and the last modified is the one will end up propagating in the Domain (used).

### Equal ConfRevNum: (at least that is the case on Packet-Tracer)
In case of both switch have equal Configuration Revision Number, and both belong to the same VTP domain, but both have different VLANs configured on them, when you connect them, None of them will exchange its database information.


when two switches have the same ConfRevNum and are part of the same domain, but with different VLAN database, you will get
`%SW_VLAN-4-VTP_USER_NOTIFICATION: VTP protocol user notification: MD5 digest checksum mismatch on receipt of equal revision summary on trunk: Gi0/0`
