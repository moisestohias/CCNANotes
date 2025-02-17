1-SpanningTreeProtocol

#STP #SpanningTreeProtocol #Cisco #Networking #IT

Read this: spanning-tree optional feature site:cisco.com filetype:pdf

## Lesson 1 Spanning Tree Protocol
Redundancy is a good thing in a network topology, if one link fails, we have a another standing by ready to take over, but if the redundant link is always forwarding traffic even if the main link is up and working, layer 2 loops can occur which are a bad thing. Fortunately STP can cleverly block redundant links, to avoid layer two topological loops.

>Note: topological loop, bridging loop mean the same thing, which is layer two loop

Historically, the IEEE first standardized STP as part of the IEEE 802.1D standard back in 1990, with pre-standard versions working even before that time (namely DEC and IBM STP variants) which was developed by Radia Perlman in the mid 80s. Over time, the industry and IEEE improved STP, with the eventual replacement of STP with an improved protocol: Rapid Spanning Tree Protocol (RSTP). The IEEE first released RSTP as amendment 802.1w and, in 2004, integrated RSTP into the 802.1D standard.

# Three Classes of Problems Caused by Not Using STP in Redundant LANs 
+ **Broadcast storms**: The forwarding of a frame repeatedly on the same links, consuming significant parts of the links' capacities
+ **Multiple frame transmission**: A side effect of looping frames in which multiple copies of one frame are delivered to the intended host, confusing the host.
+ **MAC table instability**: The continual updating of a switch's MAC address table with incorrect entries, in reaction to looping frames, resulting in frames being sent to the wrong locations

With STP or RSTP enabled, some switches block ports so that these ports do not forward frames. STP and RSTP intelligently choose which ports block, with two goals in mind:
+ All devices in a VLAN can send frames to all other devices. In other words, STP/RSTP does not block primary ports, cutting off some parts of the LAN from other parts.

|Pornt name         | definition                                                                         |
|--------------------------------------------------------------------------------------------------------|
|Root Port          | the port on **non-root bridge** that is closes to the root bridge, in term of cost.|
|Designated Port    | the port on network segment that is closes to the root bridge, in term of cost.    |
|non-Designated Port| a port that block traffic in order preserve loop-free layer 2 topology.            |
|Disabled port      | a port that is administratively shutdown.                                          |

>Note: A port that is blocked by STP means, simply that it cant learn MAC addresses, it still recieves BPDUs, broadcast..

|Pornt name         | definition                       |
|------------------------------------------------------|
|Root Port          | Forward upstread toward the root |
|Designated Port    | Facing downstread                |
|non-Designated Port| Droping the traffic              |



### How the cost is calculated and summed
The cost associated with a path in a STP is determined by summing up the costs of all the ports the frame would traverse on that path. Each switch, upon receiving a Bridge Protocol Data Unit (BPDU) on one of its ports, **incorporates the cost of that incoming port into the overall path cost calculation**. It's important to note that BPDUs themselves don't inherently start with a cost of 0 as they exit the root bridge. Instead, BPDUs contain information about the sending switch's Bridge ID, which includes its own MAC address (Burned-In Address or BIA), the path cost from the root bridge to itself, and other information. The receiving switch then uses this information to calculate the path cost from the root bridge to the sending switch, including the cost of the receiving port.

>Note: Cisco switches has long used the default settings as defined as far back as the 1998 version of the IEEE 802.1D standard. The latest IEEE standard to suggest RSTP default costs the 2018 publication of the 802.1Q standard, suggests values that are more useful when using links faster than 10 Gbps.

|Speed   |Cisco| IEEE 2004 and after |
|--------|-----|---------------------|
|10  Mbps| 100 | 2,000,000           |
|100 Mbps| 19  | 200,000             |
|1   Gbps| 4   | 20,000              |
|10  Gbps| 2   | 2000                |
|100 Gbps| N/A | 200                 |
|1   Tbps| N/A | 20                  |

>Note: Cisco Catalyst switches can be configured to use the usable IEEE cost, using the command:
spanning-tree pathcost method long 

# Bridge-ID (BID): 
The Bridge-ID is a number that switches use to elect the root a bridge on a network segment, **the device with the lowest Bridge-id, will be elected as the Root Bridge**. The Bridge-ID (8 Bytes) value is a combination of two fields, 
+ The priority field (2 Bytes) which has a default value of 32768, and can be changed to affect the election, allowed range 0 - 61440 ($2**16 - 4096$), 
>Note: The priority can be changed in increament of 4096, this is because Cisco supports PVST/Rapid-PVST it needed a mechanism to include the VLAN-ID in the BID, so it stole 12 Bits (enough to represent all extended range VLAN-IDs 0-4096) from the 2 bytes of the priority field and named them System-ID-Extension (aka Extended System-ID) as an extension the System-ID (MAC address field), that because this field can't be configured, it is only defined by the VLAN-ID. So each STP instance takes the priority field and offset it (add) by its VLAN number, to generate its unique Bridge-ID.
+ System-ID (6 Bytes) which is the base MAC address of the switch, that is showen in output of `show version` but in emulated environment such as IOU/IOSV this will be the BIA (Burned-In Address) of the first physical interface (some resources claims that this is the lowest MAC address among all the active interfaces on the switch if the first is either not active or not the lowest, it will selecte the lowest active. But I tried this, it turns out not the case, I disabled the first interface, but its mac address still is used, you must check this on a physical hardward. 

Display the Bridge ID information for the specified VLAN(s): `show spanning-tree vlan <vlan-id> | include Bridge ID` 


>Note: all switches by default has a priority value of 32768, so if multiples switches were connected with no prior configuration, the priority value is a tie, so it is up to the base MAC field to decide which device will be the root bridge.

>Note: if one switch has two or more links to the root bridge (switch) and all of them has the same speed the port with the lowest ID is going to be the Root Port

## STP Analysis Steps: (STP election process)
+  First : determine the root bridge (the switch with the lowest BID Bridge-ID). All the ports on the root switch are designated ports, because they are the closest you can get to the root bridge on any network segment.
+  Second: determine the root port on the other switches, which are the ports with the lowest cost to reach the root bridge.
+  Third : determine the Designated Port On Each Network Segment, which is the port closest to the root in term of cost, if two siwteches have the same cost, BID is the tie breaker.
+  Forth : what is left is the non-designated ports.
+  Fifth : If the switch has two paths with equal cost to reach the root bridge, it choses the path with the lowest BID of the upstream switches.
+  Sixth: if a switch have two parallel connections to an upstream switch (not configured as Ether-channel), it choses the link connecting to the upsteam swtich's port with the lowest Port Priority, if the priority is a tie then the PID (Port-Id) (ia; Int-ID) 

### STP Convergence Time
> Note: Assuming The STP Happy Triangle Topology: Figure 9-7 Official CerGuide
If the root switch's port connecting to the bottom switch fail and stops outsourcing BPUDs. The bottom switch waits for 20 seconds for BPDUs to be received from its root port -Assuming that the bottom switch didn't detect the link failure, if it did however it will directly go into the Listening State- (making sure that the root actually failed), while keeping the blocking port in the blocking state.
After the switch has been waiting for 20s, and not receiving a BPDU on what was the root port, it knows at this point something has changed and it needs to chose a different path to the root, (assuming it has a blocked port where Superior BPDUs of the root bridge is receive on it) the blocking port enters The Listening State and stays in the listening state for forward delay (default 15 seconds), during the listening state the switch is listening for BPDUs, and then the switch enters the learning state which also lasts for forwarding delay (15 second by default), where the switch populates its MAC table, but do not forward any traffic. After the learning state end finally, the switch enters the forwarding state and become the root port and start forwarding traffic.
>Note: as soon as the as MaxAge timer expires the switch flushes MAC address entries related with its old root port from its table, as they could cause temporary loops.

# STP Variant (PVST/PVST+):
The type of STP we have been discussing is the traditional STP, which uses a single STP topology (or a Common STP (CST)) for all VLANs. The drawback of this approach is inability to perform VLAN traffic engineering across redundant links: if a link is blocked, it is blocked for all VLANs.
which may be sub-optimum, and another cons is its slow convergence speed (50 seconds).
Another issue related to STP construction - more traffic is forwarded over the links closer to the root bridge, which puts higher demand on the root bridge resources - both in terms of CPU and links capacity utilization.

CISCO enhanced STP with a variant called Per VLAN STP or PVST which used for ISL trunks and PVST+ for 802.1Q trunks. Enhancing basic Layer 2 traffic engineering. Which later incorporated many features from Rapid-STP, as uplinkfast, backbone fast.
One downside of PVST/PVST+ is if we have many VLANs we need many instance of STP, one for each VLAN, CISCO came up with a solution for that is to use Multiple Instance Spanning Tree Protocol (MISTP) which was standardized as Multiple Spanning Tree Protocol (MSTP) IEEE 802.1S, in which we create multiple STP and we assign the appropriate VLANs to them based on the topology. 

>Note: PVST+ communicate the VLAN-ID in the Bridge-ID priority value field, by adding the VLAN ID into this field. So if you have the default priority value of 32768 and VLAN-ID of 100 the Bridge-ID priority value will be 32868.

# RSTP & RVST+:
Rapid STP aka RSTP (IEEE 802.1W) is the another variant of STP which was created to enhance the convergence speed of STP, and CISCO has its own version of this which is integrated to PVST+ (Per VLAN STP) and it's called Rapid PVST+.
-One of the ways used to make RSTP/Rapid PVST+ fast in term of convergence, is that the switch that experience topology change like one of its links goes from the blocking state to the forwarding state, will notify the other switches in the topology by setting a bit called TC (Topology Change) bit, in the PBDU, and this can cause the other switch to flush their MAC address table and relearn.

>Note: by default CISCO switches will be running PVST+ as the STP implementation. And if we don't configure or change anything to change the STP topology, PVST+ will work just like traditional STP, and come up with the same STP topology.

>Note: The addition or removal of VLANs when STP runs in per-VLAN spanning tree (PVST / PVST+) mode triggers spanning tree recalculation for that VLAN instance and the traffic is disrupted only for that VLAN. The other VLAN parts of a trunk link can forward traffic normally. The addition or removal of VLANs for a Multiple Spanning Tree (MST) instance that exists triggers spanning tree recalculation for that instance and traffic is disrupted for all the VLAN parts of that MST instance.

# STP Verification And Troubleshooting Commands:
  show spanning-tree
  ! display the STP status for all STP instances (VLANs)
  show spanning-tree active
  ! display active STP instances status for all VLANs, same as 'show spanning-tree'
  show spanning-tree detail
  ! display detailed STP status info for all VLANs, and thier trunk ports
  show spanning-tree vlan "VLAN ID"
  ! display the STP status for the specified VLAN
  show spanning-tree interface "int ID" 
  ! displays STP status for each VLAN on the specified Interface
  show spanning-tree summary
  ! displays the status of the: Ext-sys-ID, Portfast, BPDU Guard, BPDU Filter, Loopguard 
  show spanning-tree summary total
  ! same without total, but only display the total

Example: show spanning-tree
VLAN0100 is executing the ieee compatible Spanning Tree Protocol
  Bridge Identifier has priority of 32768, sysid 100, 000B.BECD.4301
  Configured hello time 2, max age 20, forward delay 15
  Current root has priority 32868
  Root port is 25 (GigabitEthernet0/1), cost of root path is 4
  Topology change flag not set, detected flag not set
  Number of topology changes 0 last change occurred 00:00:00 ago
          from FastEthernet0/1
  Times:  hold 1, topology change 35, notification 2
      hello 2, max age 20, forward delay 15
  Timers: hello 0, topology change 0, notification 0, aging 300

Port 1 (FastEthernet0/1) of VLAN0100 is designated forwarding
  Port path cost 19, Port priority 128, Port Identifier 128.1
  Designated root has priority 32868, address 0003.E491.807A
  Designated bridge has priority 32868, address 000B.BECD.4301
  Designated port id is 128.1, designated path cost 19
  Timers: message age 16, forward delay 0, hold 0
  Number of transitions to forwarding state: 1
  Link type is point-to-point by default

Port 24 (FastEthernet0/24) of VLAN0100 is alternate blocking
  Port path cost 19, Port priority 128, Port Identifier 128.24
  Designated root has priority 32868, address 0003.E491.807A
  Designated bridge has priority 32868, address 0003.E491.807A
  Timers: message age 16, forward delay 0, hold 0
  Number of transitions to forwarding state: 1
  Link type is point-to-point by default

Port 25 (GigabitEthernet0/1) of VLAN0100 is root forwarding
  Port path cost 4, Port priority 128, Port Identifier 128.25
  Designated root has priority 32868, address 0003.E491.807A
  Designated bridge has priority 32868, address 0003.E491.807A
  Timers: message age 16, forward delay 0, hold 0
  Number of transitions to forwarding state: 1
  Link type is point-to-point by default

# STP configuration:
spanning-tree vlan "vlan id" root primary: set the switch to be the root bridge for the specified VLAN. 
spanning-tree vlan "vlan id" root secondary: set the switch to be the root bridge for the specified VLAN in case of if the primary root bridge fail. 


### portfast and PBDU-Guard:
By default, all of the switch's ports are set to dynamic auto, which causes them to passively wait for another switch to initiate the negotiation of the formation of a Trunk that they might be connected to, and if no device initiate the negotiation, they will stay operating as an access port, because they don't know whether they are connected to a switch or end device (they play nice).
But once the switch boot, it doesn't know whether it's connected to another switch or end host, STP wants to make sure that no layer 2 topological loops are present so it puts all ports in Listening State (15s), then Learning state (15s), and together they take 30 seconds, which something that can cause a problem in case of a port connecting to end device that might be initiating DHCP Discover, which causes it to fail and assume that there is no DHCP server. Luckily, CISCO has a function called portfast which resolves this and other problems as well.

-portfast is feature in CISCO switches that tells the switch to disable STP learning, and listening states for the specified port. In another word basically telling the switch that this port will be connected to an end device, and BPDU should never be received off of this port.
SW#(config-if)spanning-tree portfast

-BPDU Guard: place the switch port configured for portfast in ErrDisable state if a BPDU is received on that port. So no one can plug its switch and mess the STP topology.
SW#(config-if)spanning-tree bpduguard enable

#### configuring portfast and bpduguared on all access ports from the global configuration.
spanning-tree portfast default
! enables port fast on all access ports
spanning-tree portfast bpduguared default
! enables bpduguared on all access ports, and are set for portfast

Another benefit of PortFast is that Topology Change Notification (TCN) BPDUs are not sent when a switch port in PortFast mode goes up or down. This simplifies the TCN transmission on a large network when end-user workstations are coming up or shutting down.

>Tip: You can also use a macro configuration command to force a switch port to support a single host. The following command enables STP PortFast, sets the port to access mode, and disables PAgP to prevent the port from participating in an EtherChannel:
SW(config-if)# switchport host
! switchport mode will be set to access
! spanning-tree portfast will be enabled
! channel group will be disabled


To summarize, if a port is configured as portfast and bpduguard, this will disable STP learning and listening states, and if BPDU is ever received put the port in ErrDisable state, and to recover from ErrDisable state the administrator need to resolve the issue, and re-bounce the interface (shut, no shut).



! IOSvL2 Switches
spanning-tree portfast edge default 
spanning-tree portfast edge bpduguard default 

spanning-tree portfast network default 
spanning-tree portfast network bpduguard default 

spanning-tree normal

- PORTFAST EDGE is used to configure a port on which an end device is connected, such as a PC. All ports directly connected to end devices cannot create bridging loops in the network. Therefore, the edge port directly transitions to the forwarding state, and skips the listening and learning stages. However, the specific command configures a port such that if it receives a BPDU, it immediately loses its edge port status and becomes a normal spanning-tree port.
-You use the PORTFAST NETWORK on trunk ports to enable bridge assurance feature which protects against loops by detecting unidirectional links in the STP topology. But normally bridge assurance is enabled by default.
A spanning tree NORMAL port is one that functions in the default manner for spanning tree. Under normal circumstances it will transition from the Listening, Learning, Forwarding stages based on the default timers.


[From CISC community](https://community.cisco.com/t5/switching/spanning-tree-portfast-default/td-p/1937659)

It can be quite dangerous to enable portfast on a trunk because often trunks are switch-to-switch. That is why enabling portfast globally usually goes hand-in-hand with spanning-tree portfast bpduguard default, which restores some level of protection against loops.

If you really want portfast on a trunk, which you might do if the trunk is connected to a server, then you have to put 'spanning-tree portfast trunk' on the interface itself. There is no way to enable portfast on trunks globally, precisely because of the safety concerns against loops.

when you enable portfast globally you will get the following message:
%Warning: this command enables portfast by default on all interfaces. You should now disable portfast explicitly on switched ports leading to hubs, switches and bridges as they may create temporary bridging loops.

It is slightly misleading, the spanning-tree portfast default activates PortFast only on ports in the access operational mode. If the port is operating as a trunk, this command does not apply to it. While the message says "on all interfaces", it is simply being too general Some should really review these system messages in IOS once in a while and correct inaccuracies, like this one.

### spanning-tree portfast trunk VS spanning-tree portfast
The interface-level command spanning-tree portfast will be effective only if the port is operating as an access port. The spanning-tree portfast trunk activates PortFast on the port regardless of its operating mode (access or trunk).


#### From CISCO website
Before you configure STP, select a switch to be the root bridge of the spanning tree. This switch does not need to be the most powerful switch, but choose the most centralized switch on the network. Also, choose the least disturbed switch in the network (in term of topology changes). The backbone switches often serve as the spanning tree root because these switches typically do not connect to end stations. Also, moves and changes within the network are less likely to affect these switches.

After you decide on the root switch, set the appropriate configuration to designate the switch as the root switch. The only variable that you must set is the bridge priority. If the switch has a bridge priority that is lower than all the other switches, the other switches automatically select the switch as the root switch.


# Steady STP state:
- The root switch outsource BPDUs from all of its designated interfaces. listing itself as the root bridge and the cost of 0.
- each non-root switch receives the BUPD of the root bridge from their root port, adds the cost of its root port (receiving port), and list its BID as the sender's BID.
- These steps repeat till something change.

When a switch doesn't receive a Hello, it knows a problem might be occurring in the network. Each switch relies on these periodically received Hellos from the root as a way to know that its path to the root is still working.
Common-STP/PVST reacts a bit differently than RSTP/RPVST/MST, but they all rely on the MaxAge timer 
MaxAge (default: 10 times the Hello (20 sec)): How long any switch should wait, after ceasing to hear Hellos, before trying to change the STP topology.
If if the MaxAge expires and no BPDU has been received, the election process take place again, with   every switch start again by claiming itself as the root bridge.

If its a direct link failure (local port has been shutdown or failed) the switch doesn't wait the MaxAge timer to expire, instead it immediately start

>Note: If the root switch fail (STP process fail/halt) which could happen and stop outsourcing BPUDs. All non-root switches wait for 20 seconds for BPDUs to be received from their root port, (making sure that the root actually failed), while keeping the blocking port in the blocking state.

# Port Role vs Port State
STP uses the idea of roles and states. Roles, like root port and designated port, relate to how STP analyzes the LAN topology. States, like forwarding and blocking, tell a switch whether to send or receive frames. When STP converges, a switch chooses new port roles, and the port roles determine the state (forwarding or blocking).

>Note: RSTP was published first in 2001 as IEEE 802.1w. As it stands today (after 2011), RSTP actually sits in the 802.1Q standards document, but most people still referring to it as 802.1w.

