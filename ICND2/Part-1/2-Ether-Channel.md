2-Ether-Channel.md
#Cisco #Networking #IT
# Ether channel:
Ether channel lets us logically bundle multiple links and use them as one link or as better known Port channel, which allows us to get higher bandwidth, and load balancing between those links, and the cherry on top, if one link goes down the other ports will continue to operate normally.

>Warning: always use dynamic protocol (LACP or PAgP) avoid using static on, because if one of the link fail the check or it is not part of the Channel anymore, the dynamic protocol will detect that, it dose this by sending keepalive messages between the ports of the link partner that are member of channel.
From the switch perspective any ports that's configured as on, is considered as a member of the channel even if it is really not due to unmatched configuration with the link partner. 
The "on" side will use just one of the aggregated port to send STP BPUDs but not the rest, so the other side of the link is not recieving duplicate BPUDs on the other port, so it won't block them (it's not aggregating them either) the problem occur when a broadcast message is fired, the "on" side will send the broadcast from only a single port of the channel, the not configured side will send it out of all ports, creating a loop.
!Note: Even in the absence of broacast messages, the loop still could occur. If the traffic comes from behind and load-distribution is configured to anything other than the source-mac, from the not aggregating side perspective the MAC is flapping. 

# EtherChannel misconfiguration guard:
It's feature by Cisco made specifically to combat the problem of the ether-channel "on" static configuration, on unmatched port configuration, which check if the all configured members are active in the same ether-channel, otherwise it enable STP for unactive ports. It is on by default on most IOS version. but you can always enable it : "spanning-tree etherchannel guard misconfig" or disabled it by appending no if you want to delibrately create loops.
!Note: This feature doens't always work, for what ever reason.

# Ether channel Benefits:
+ Higher bandwidth between switches
+ Provide load balancing
+ Redundant links

CISCO was the first to come up with this idea of port channel and called it Ether channel, and the protocol that does the dynamic port aggregation is called PAgP (Port Aggregation Protocol), then the industry catched up with the IEEE  802.3AD LACP (Link Aggregation Control Protocol).
  
### Load Balancing Mechanism:
The machinery used to split the traffic between the different links is kind of rudimentary, it forwards the traffic over specific link based on the MAC or IP (modern switches provide "port" option as well) of either the source or destination, or both.
If you have TWO links that are making up the Port-Channel, you need one bit to address those links, and whatever the last bit of the packet matches, that is the link that will be used to forward the traffic over. 
If you have four links that are making up you Port Channel, you need two bits to address them, and the same story with two links.

>Warning: For either dynamic protocols and the "on" mode, used to create PortChannle, all the links that make up this port channel should should have the same setting -the whole list of settings that need to match and other details are in the other documents-.

### Modes:
+ ON: Tells the port to be part of Ether-channel, but it doesn't exchange negotiation messages.
+ Auto/Passive: passively waits for the PAgP/LACP frame to be received if so, respond back.
+ Desirable/Active: initiate the negotiation of the Ether-channel.

### PAgP Negotiation
  | PAgP mode | on  | auto  | Desirable | 
  |-----------|-----|-------|-----------|
  |   On      | Yes |  No   |    No     |
  |  Auto     | No  |  No   |   Yes     |
  | Desirable | No  |  Yes  |   Yes     |

### LACP Negotiation
  | PACP mode | on  | auto  | Desirable | 
  |-----------|-----|-------|-----------|
  |   On      | Yes |  No   |    No     |
  | Passive   | No  |  No   |   Yes     |
  |  Active   | No  |  Yes  |   Yes     | 

>Warning: Notice in the two table above, "on" mode doesn't work with either dynamic protocol (PAgP nor LACP) because it doesn't participate in the negotiation, the dynamic side never hears back from the ON side. 
!Note: When you configure certain ports of a switch with "On" mode to be as part of an ether-channel, it just statically assume all ports to be as part of the channel and it doesn't check with its link partner.

## Ether Channel Configuration Commands:
inter range g0/0 - 1
shutdown !It's always recommended to shut down the interface first, so no temporary loop occurs.
! Conifugure the settings that need to be matched first
channel-group <number> mode on/desirable/active
!Note: the number is only locally significant, doesn't have to match with the link partner.
!This command will create a logical interface for the Channel-Group, which you can go a head a configure as a trunk.
!Note: if you don't down the interface while configuring the port-channel, and it concluded that a loop occured, it will put all phyisical member as well as the logical port-channel in error-disable state, in that case you need to flap the logical port-channel (it will flap the physical members), THE LOGICAL PORT-CHANNEL MUST BE FLAPPED.

int port-channel 1 
switchport encapsulation dot1q
switchport mode trunk
>Warning: Any command issued on the port-channel will be executed on the physical members (except for he "no switchport"). But it's best to run these command on the physical members using range command (or by shutting the Port-Group first), because sometime (this happens often) when you run : "switchport encapsulation dot1q" and "switchport mode trunk" consecutively the switch will apply the command on the first physical member then move to next, which is a process that may take a sec, in the mean time LACP/PAgP could initiate its check, and conclude the first the member do not have matching settings with the rest of the group, thus causing transient loops in the data plane.

!Note: when you configure certain ports on a switch to be LACP/PAgP and the link partner didn't respond to negotiation frame, the channel will be suspended and the links will be used independently as normal independent links, thus ports will show in the "show etherchannel summary" (I) next to them. At a later time when the link partner respond and the channel is formed the will be part of the channel (not independent anymore) and they will (P) indicated they are bundled.

# Changing the load balancing algorithm:
port-channel load-balancing ?
! dst-ip       Dst IP Addr
! dst-mac      Dst Mac Addr
! src-dst-ip   Src XOR Dst IP Addr
! src-dst-mac  Src XOR Dst Mac Addr
! src-ip       Src IP Addr
! src-mac      Src Mac Addr
port-channel load-balance src-dst-ip
! the result of the XOR operation between the source and destination IP of the packet will determine the link, and this will introduce some randomness to which link the traffic will go over

#  Why you must use LACP or PAgP:
When configuring the physical members of the EtherChannel as "on", if the settings of one of those port didn't not match the other members that port is not added to the group, but still used thus causing a loop.
But when using dynamic protocol to negotiate the EtherChannel formation, if a given physical link passes the check, the link is added to the EtherChannel and used; if not, it is placed in a DOWN STATE, AND NOT USED (No Loops), until the configuration inconsistency can be resolved.

>Warning: AP (Access Point) don't use dynamic aggregation protocol, they can be on or off.

#  Layer three Ether-channel: (L3 Ether-Channel)
>Warning: the order of configuration matters in the case layer three channel, since the channel will inherit the state of its members the moment the channel is created, you must configure the physical members first with the correct settings that you want for the channel, then create the channel (the physical members have to be converted to layer three interfaces before converting the channel itself).
The only additional configuration is to configure the switchports that are member of the channel as "no switchport" and the port-channel it self as well (even though the port-channel will inherit the switchport state from the physical members, but it doesn't hurt to be sure), and assign the port channel and IP address, no shut, you golding.

#  Layer 3 vs Layer 3 Ether-channel:
LAG is independent of the port mode. It doesn't matter whether EtherChannel is an access, trunk, or layer 3 port, from LACP/PAgP perspective they all the same.

#  The difference between running layer three ether-channel vs multiple independent layer three interface forming independent adjacency?
The load balancing would be based on the link utilization as its carried out by CEF when running multiple independent layer three ports, versus of the static layer two load distribution.
Something else you might be concerned with is that now we pushing more load on the layer three control plain, as suppose layer two (which is usually native and more performent).
Something else to note when running layer three ether-channel, is when one of the member of the channel goes down, this will not cause a RP re-convergence.

#  Protecting again misconfiguration:
There are multiple protection counter-measure to protect against misconfiguration:
# You may intend to run LACP every where, but still make mistake of issuing active/passive on the interface, and to protect again that, you can use: "channel-protocol lacp", on the interfaces, which doesn't dictate what the protocol is, but it dictate what the channel protocol can be. 
# To protect spanning-tree loops, you can use "spanning-tree ether-channel misconfig guard" which is enabled by default on most switch.

# Ether Channel Verification And Troubleshooting Commands:
show inter port-channel "Int ID"
show etherchannel summary
! displays summary info about the Ether channels on the switch, their ports, protocol.
show etherchannel port-channel
! displays info about the port channel, protocol, ports. state.
show etherchannel load-balance
!displays the load balancing approach each Ether channel is using.
show lacp neighbor
!display LACP neighbor port's attributes, when using LACP.


DSL-2750U 

The typical use of Ether-channels is aggregating ports from one switch to another switche. But for added resiliesy toward device failure and not just a link failure while maintaining bandwidth, multi-chassis Ether-Channel technologies are used, such as vPC (virtual port channel) on the Nexus platform (other vendors have equivalents, such as Force 10’s VLT), or other TRILL-based technology.

# Multi-chassis EtherChannel (aka M-Lag):
Is the concept of aggregating physical ports from multiple switches into a single etherchannel, to overcome the problem of fate sharing, and obtain the benefits of Ether-channel: Load-Distribution and Resiliency.
!Fate sharing: if an access layer switch fail, PRACTICALLY all connected end devices fail. For instance even if your servers have redundant power supply, redundant NICs, if they are connected to same access switch, and that switch fail, the servers will share the fate of that switch.
!Note: you can overcome the problem of fate sharing if you have two NICs connected to different switches, but you won't be able to use both of them simultaneously, one of them will be in a standby mode.
!Note: with M-Channel from the hypervisor perspective the EtherChannel is connected to a single physical switch, so it distribute the load however the load distribution algorithm define.

Because the Multi-chassis EtherChannel is so complicated to implement since you need somehow a way to aggregate the control plane of separate physical boxes, in fact even different products from the same company like Cisco will not interoperate with each others, like Cisco for instance which has separate technologies for different product line.
