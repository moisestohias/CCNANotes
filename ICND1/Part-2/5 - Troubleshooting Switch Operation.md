# 4 - Troubleshooting Switch Operation

As a network engineer you will responsible for many technologies, not just understanding their basic concept of operation or configuring them, but most of the time maintaining them making sure everything is in place working correctly as expected, and many time troubleshooting them when something don't operate (or at least as expected). You have to be ready to predict what symptoms would occur when the network was misconfigured in particular ways. What symptoms should happen and how to recognize those problems.
When things aren't working as expected, we need to perform Troubleshooting, by isolation the problem and narrow the scope of what might be the issue.

# Lesson 1 Isolating the Issue
Ping: the ping command is the first tool at your disposal to check layer 3 reachablity between device and it's available in most devices (PCs, Switches, Routers). It reports the min, max and the average time took to perform the ping, it support multiple options:
+ `repeat`: the number of ping that'll be sent
+ `timeout`: the time of second before it assume the ping fail
Traceroute: is the second most hand tool for troubleshooting routing protocols decisions, and it's available in most devices (PCs, Switches, Routers).

# Lesson 2 Checking Interface Status
+ `show ip int brief`: displays consolidated info about the ports
+ `show int <G0/1>`: display detailed info about the interface, like interface status (up or down), bia, BW, duplex, and statistical info.

### Interpreting output from the show interface command
Interface Status (up or down), at the :
physical-layer : up, if it is receiving Carrier Detect Signal.
data link-layer: up, if it is receiving Keep Alive from the other end.

### Interpreting output from the show interface command ON THE SWITCH:
+ `up: up(connected)` > functional
+ `up: down` > connection issue(layer 2 keep alive not received)
+ `down: down(not connected)` > the cable not connected at least to one end, or the other interface is administratively down.
+ `down: down` > layer 1 interface issue, or shutdown after violation has occured.
+ `admin-down down`: interface administratively disabled.

>Note: in the router if an interface was brought up using "no shutdown" command and assigned IP address, but no device is connected to that port, or the other device interface is administratively down, the status of that interface, will be UP/DOWN.

show int status : display the operational status of all interfaces status, duplex, speed and the interface type in one screen, "a-" prefix means the setting was auto-negotiated.
show int status err-disable: displays only the err-disabled interfaces and the cause.

speed and duplex mode mismatch is common reason for connectivity issues, when one side is administratively set for full-duplex and the other side to auto-negotiate, the administratively-set side won't participate in the negotiation of duplex mode, and the other side will end up defaulting to half-duplex

# Lesson 3 Checking for Interface Errors
### Common Layer 1 Problems on Working Interfaces:
When the interface reaches the connect (up/up) state, the switch considers the interface to be working. The switch, of course, tries to use the interface, and at the same time, the switch keeps various interface counters. These interface counters can help identify problems that can occur even though the interface is in a connect state. This section explains some of the related concepts and a few of the most common problems.

- **Runts** : Frame that's too small (less than 64 B) (64 bytes, including the 18-byte destination MAC, source MAC, type, and FCS). Can be caused by collisions., And has a bad CRC.
- **Giant** : Frame that's too big(bigger than 1518B) (1518 bytes, including the 18-byte destination MAC, source MAC, type, and FCS)., And has a bad CRC.
>Note: the bad CRC occurs because the CRC implementation on Ethernet is designed to operate for frames that are between 64 - 1518 byte long.

- **CRC** : FSC mismatch.
- **No buffer** : This means you don’t have any buffer room left for incoming packets. Any packets received once the buffers are full are discarded. You can see how many packets are dropped with the ignored output.
- **Ignored** : If the packet buffers are full, packets will be dropped. You see this increment along with the no buffer output. Typically if the 'no buffer' and 'ignored' outputs are incrementing, you have some sort of broadcast storm on your LAN. This can be caused by a bad NIC or even a bad network design.
- Frame overrun: This output increments when the frames received are of an illegal format, or not complete can be caused by collisions, which is typically incremented when a collision occurs.

- **Collision** : the total number of collusion occurred.
- **Late Collision** : The subset of all collisions that happen after the 64th byte of the frame has been transmitted. (In a properly working Ethernet LAN, collisions should occur within the first 64 bytes; late collisions today often point to a duplex mismatch).
"late": taking to long to travel the medium, aka greater than the medium slot time, (medium-slot-time is twice the time required for a bit to transit the maximum distance of the link ), later than 51.2 micro-second for 10 Mbps, 5.15 micro-second for 100 Mbps, 4.096 for 1Gbps. (Personally i don't know how these timers are calculated)

- **input errors** : sum of all errors on the interface: runts, giants, no buffer, CRC, frame, overrun, and ignored counts.
- **output errors** : total number of errors that prevented the frame to transit

>Note: Duplex mismatch is one of the main reason for slow link.
>Warning: Gig to Ethernet link, will fail the duplex negotiation, so you need to hardcode the duplex in such scenario.
>Note: The negotiation of the duplex will occur every minut, if you don't manually configure it it will fail, and log the following to the consol.
*Nov 26 07:54:43.070: %CDP-4-DUPLEX_MISMATCH: duplex mismatch discovered on GigabitEthernet0/2 (not half duplex), with IOU2 Ethernet0/0 (half duplex).
>Note: On gig interface usually they are configured with "negotiation auto" to negotiate all layer 1 settings (mdix, speed, duplex)

>Note: when one side of the link is reporting zero or few CRC errors, and many late collision but the other side contrarily, this is a textbook indication that there's duplex mismatch, the side with hight CRC errors that's th full-duplex and the other side with late-collision that's the half-duplex **
>Note: when you only see input errors, without runts, giants, or throttles, this might occur when one side of the link is configured as no switchport, but the other side is not, so switchport side keep sending the DTP negotiation packets.
!TIP: runts, occur in the case of duplex mismatch, the half duplex side, stops sending currently outgoing frames immediately when receiving frames from the other side, so the full duplex frame receives bad CRC and runts occasionally.

### Clearing The Interface Error Counters:
After resolving the what ever was causing the problem in your link, you might want to reset the interface counters to zero, so next time you won't be fooled by the old errors.
`#clear counters` ! user exec command! clears the counters for all interfaces.

# Lesson 4 Discovering Neighbors
- CDP (CISCO Discovery Protocol): is a CISCO proprietary protocol that allows CISCO devices to discover other CISCO neighboring devices at layer 2, Note : CDP doesn't use IP address, it happens at layer 2.

> cpd run : enable the CDP globally
> no cpd run : disable the CDP globally
>(config-if) no cpd enable : disable CDP for specific interface

> show cdp: shows CDP settings: timers, and the version.
>Note: we mainly use CDP v2
- CDP by default send packet every 60s
- CDP hold-time by default is 180s (time before the neighboring device is considered dead)
>Note: we mainly use CDP v2
> no cdp advertise-v2: run CDP version 1 instead of v2

> show cpd neighbors : display CISCO neighboring devices
> show cpd neighbors detail: display detailed info about each neighboring devices

>Note: VLAN Hopping: when two or more switches doesn't agree on the same Native Vlan. **



