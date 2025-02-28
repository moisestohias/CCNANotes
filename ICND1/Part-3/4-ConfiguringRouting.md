<!-- 4-ConfiguringRouting.md -->

# Dynamic Routing Protocols
+ Vector: has a magnitude and direction
+ Distance Vector RP: has a magnitude(Distance) and direction (next hope).  

# Routing Information Protocol (RIP)
+ RIP uses a variant of the Bellman-Ford Algorithm (Bull shit). 
+ Each router calculates the distance (Hop count) to other networks we know about, then every 30 s each router tells their neighbors (broadcast) about all networks he know about.
- the RIP's metric is the Hop Count, which is the number of hops (routers) that the packet need to traverse to reach its distination, 
+ The maximum Hop Count is 15, which limits how large the network could be
+ limiting the Metric to only the Hop Count, make the RIP less reliable, bc it's not considering the Bandwidth, and other link variables...
+ the metric doesn't concern it self with anything other than Hop Count which make, limited view 
+ when a RIP learn about network it'll add it to its Routing Table, and broadcast that newlly learn info to all of its neighbors except the interface that it learned it from , and this known as Split Horizon.


# EIGRP (Enhanced Interior Gateway Routing Protocol) : 
Is an advanced DV RP
- Uses DUAL (diffusing Update Algorithm)
- The router is going to form neighborships with adjacent EIGRP speaking routers, they will initially exchange their full routing table. but after then they will just updates   
- Each device advertise it's metric (distance) to all network it know about, and it's called the Advertised Distance : which is the distance from an EIGRP to specific network.
- Once the other router take that advertises distance and add its distance to that router that it got this information from, this called the Feasible Distance.
- Than the router it take all the its feasible distance to the same network and pick the best (smallest), and this called the Feasible Route, and the second path is called the Successor Route.
- Feasible Route : primary path to a destination network, based on the best feasible distance.
- Successor Route: a back up path to a destination network, based on the second best feasible distance.   

## Link State RP+
+ **OSPF** (Open Shortest Path First) uses Dijkstra Algorithm, v2 relies on IPv4 to carry routing info, and v3 relies on IPv6.  
+ **ISIS** (Intermediate System to Intermediate System) also uses Dijkstra Algorithm, doesn't rely on IP to carry routing info.  

When we give the network command in the router configuration mode, we're not saying advertise this network we are saying any interface that belong to this address space to participate in the forming of neighborship and sending hello packet.

+ **Passive Interfaces**: are the interfaces that's not going to participate in the forming of neighborships, don't send Hello messages, but its address it's still going to be advertised so other Routers know about it.
+ `passive-interface gig 0/0` , set the specified interface to be passive.
+ `passive-interface default` , set all interfaces to be passive, then we can make exceptions, that make sense when the router connected to so many end user subnets, and only one or tow int connected to other routers.
+ `no passive-interface gig 0/0` , set the specified interface to be active and send hello messages.


## Configuring OSPF :
### Basics of OSPF :
> Note: it's best to think of OSPF like a map, every area map out every path to all network with its metric
-OSPF divides the network to Areas, and there's always a Back-Bone-Area that all other areas must connect to -because it's responsible for flooding routing infomation between regions- if there are other areas that are not physically adjacent to the Back-Bone-Area we can create Virtual Link.  
- OSPF Network Types : we can have different type of OSPF Network on different type of interfaces like : Frame Relay, PPP, Ethernet , we can send interface type by default.

+ OSPF is an open standard, developed by IETF (Internet Engineer Task Force) 
+ OSPF doesn't only send and receive net path information to and from it's neighbors, OSPF Establishes adjacencies with other routers running OSPF, so it maps every link and all network so all devices in the same Area have the same map (paths info).
# OSPF send LSAs (Link State Advertisement) to other routers in an area, then OSPF construct link state database from received LSAs.
**LASs are sent in LSUs (Link State Updates)** 
+ then each devices going run the Dijkstra SPF (Shortest Path First) algorithm to determine the best path, comming up with Sortest Path Tree (SPT), then that path is going to be CANDIDATE to be injected in the routing table. 
+ CANDIDATE : means just bc that OSPF came up with this route to get to this network doesn't mean it's worthy to be put in the routing table because we still have the Administrative Distance, bc if the same network has been learned by another Routing Source (4x: static) the OSPF path will not be injected in the Routing Table.

### Dijkstra : 
+ Considers a Topology and calculate the shortest Path to a destination.
### Hello: 
Protocol used to discover OSPF neighbor and confirm reachability (is the link  still alive, if we don't hear from the neighbor after certain period of time called the Dead Interval (40s) then the router can conclude its neighbor router is no longer up), also use in the election of Designated Router,

**the Hello protocol is what's going to be used for the election of the Designated Router**

## Neighborship vs Adjacency in the case of OSPF : 
### Neighbors :
- reside on the same network link (same subnet), 
- exchange Hello messages   
### Adjacencent : 
- are neighbors.
- the exchange link state updates (LSUs), and Data Description (DD) packet.

### DataBase Description packet (DBD): 
Contains all known LSUs, which are used to construct identical database between two neighbors.

# Link state Advertisement LSAs: info router sent & receive about network reachablity, and used to construct Link State data-base.   
# Link state Update LSUs: packet that carries LSAs, 
# Link state Request LSRs: used to ask for the messing LSAs, 
# Link state Acknowledgment LSAck: used by router to confirm it received and LSU.

# Designated Router DR, Backup Designated Router BDR.
- if we have many routers in the same subnet we don't have to form adjacency between every routers bc this will be redundant, routers only form adjacency with Designated and Backup Designated Routers. 

The formula for counting the number of neighborship in the case of multiple routers: 
$$ \text{Number of adjacencies} = (n*(n-1))/2 $$ 
where n is the number of router  ** same as # of link in meshed topo.

## Electing Designated Router 
The Hello protocol is what's going to be used, a router with OSPF highest priority value win the election, 
+ OSPF priority value is associated with an interface and can be a value in the range 0 - 255 *
+ OSPF with priority value of 0 means that the router will not become a DR *
+ by defaul an interface have a priority value of 1 * 

+ `(config-if)# ip ospf priority "value"` Administratively influence who's going to be the DR and BDR.

If we connect our router and all router have the same OSPF priority value, so everything is a tie, the Router with the highest Router ID (RID)

> [!Note] the RID is used to identify the node in the OSPF graph (map).

# (router)#router-id "id", configuring the Router ID
+ If the RID is not configured (using the command "router-id <RID>" command) on the routers, the router with the highest numerical IP address of the LoopBack interface that is up and operational in that router, and is going to be the DR, if no Loopback is configured then the highest numerical IP address of physical interface that is up and operational    
+ We can assign an IP from the public space to the Loopback interface, and it's not going to be advertised, and it's not reachable by the neighboring routers, and this IP is just for setting the router ID, many Net Admin do that 
+ If there's no Loopback interface up and operational, the highest IP of one of the none Loopback interfaces of that router is going to be the Router ID  

> Note: When you configure a router with OSPF it start sending Hellos as soon as you issue the first network command, if it can't find any neighbor for dead timer, it elect itself as the designated and list it self as the designated router in the Hello packet and if any new router even with a higher priority or RID, come up and find that the DR (or DR&BDR) already has/have been elected it will submit to that, and it is not going to cause it to preempt the existent DR. That's why the designated election is referred to as not-preemptive unlike STP.

!TIP: to cause the election to take place again, you can clear the OSPF process on the routers; using the command: clear ip ospf <PID> process 
> Note: "clear ip ospf proc" restarts the process and all OSPF neighbors are restarted

> Note: Even if the a router elect itself as the DR in the absence of other routers, this is only applicable for the adjacency (with whom routers form adjacency). For the master and slave election in the extart state the router with the highest RID is who will be the Master and set the starting sequence number.


# Forming an Adjacency : between two Routers R1 & R2 ===================
### 1 Down State:
- if the link is down (there's no communication between R1 and R2), both routers are in Down State.
### 2 Init State:
- when the link goes up, R1 send Hello messages (which include its RD) to R2 using the multicast IP address  224.0.0.5, and become in the Init State.
- R2 receive the Hello of R1, and it notice that its router ID is not included, so it send its router ID with R1 ID that it just learned,  and become in the Init State,

** Note: 224.0.0.5 is the multicast address to all routers **

### 3 Two Way State:
- R1 receives the hello message of R2 and see its ID listed, so it enter the Two Way State, then send another Hello message this time with R2 ID.
- R2 receive the second hello message of R1 and see its ID listed, so it enter the Two Way State it self.
** while both routers in the two way state, that's where the Designated and Back-up Designated router election takes place if needed ** 
 
### 4 ExStart State:
** if R1&R2 want to become adjacent, there are going to exchange data-base information and for this exchange process they need to decide which one gonna be the primary (send first) which one is the secondary, and this election take place in the Exstart State, and it is the Router with the higher ID that become the primary and it'll start the exchange of the data-base, ** 

### 5 Exchange State:
- when the exchange of data-base start they are in the Exchange State. 
- the primary start by sending the Data-base Description Packet.
- the secondary router receive the Data-base Description Packet info and send back an acknowledgment to inform the primary router that it got its info.   
> Note: the DataBase Description Packets contain just discription of the known LSAs

### 6 Loading State:
- when one router don't find some LSAs, it's going to ask its neighbors for the messing ones.

### Full State:
- when both router have an identical Link State data-base we've know reached fully formed Adjacency

---

### OSPF Cost (metric) 
> [!Note] OSPF uses default reference bandwidth of 100Mbps 100_000_00 bps to calculate its cost. 
Cost =  reference BW / interface BW

+ When the link is 1Gbps and the reference bandwidth is 100Mbps the cost comes out to be 0.1 and this invalid cost bc ** the cost can only be an integer of 16 bit long **, so the cost in this case is considered as 1 *
+ To fix the issue of floating point cost we use higher reference bandwidth *
> [!Warning] The problem of the reference bandwidth might show up in the adverse scenario, in case of if we set it to very high value, and the are slow link on the network like 64 Kbit/s wan link and the reference bandwidth is e.g 10G and if you do the math this will comes up to be 10,000,000 / 64 = 156,250 and this number is definitely greater than the maximum reference bandwidth 65,536 as a result the 64k links will have the same cost as 128k links.
> the reference bandwidth is a matter of balance  

+ The unit in Mbps `auto-cost bandwidth-reference "value"`

### Area: 
Ares is a collection of networks that are running OSPF, the  interfaces of the router are what belong to a certain area not the router it self as a matter of fact the router could have different interfaces that belong to different areas.

- all routers in the same area have the same view of the network they see the same network map so they run the DIGKSTRA algorithm on the same data, in a larger network we might want to divide the network to smaller areas so it dosent take to long for the router to perform its calculations especially for old routers that doesn't have much processing power, and all those areas need to be adjacent to AREA 0 aka the Back-Bone Area, router that sits in between areas and connect them is known as Area Border Router (ABR) and this router will have the data base of both areas and run the DIJKSTRA algorithm on both area
- if we have area that is not adjacent to area 0 we can creat a Virtual Link to area 0.

#### Network types:
##### Broadcast Network: 
Like in case of Ethernet, multitude of router could be connected via a switch to each others, 
- the DR and the BDR will be elected. 
- the default hello interval is 10 second.
- the dead interval is 4 time the hello interval so it's 40 second.

##### Point-to-Point Network : 
- since we only have two router, there is no need to elect DR & BDR
- the default hello interval is 10 second.
- the dead interval is 4 time the hello interval so it's 40 second.

##### Non-Broadcast Network aka NBMA(Non-Broadcast Multi Access) : 
Like frame relay network
- the DR and the BDR should be elected, but we have to strategically affect which one that will be elected as the designated. 
- the default hello interval is 30 second.
- the dead interval is 4 time the hello interval so it's 120s or 2m
- since this is not a broadcast medium the router wont be able to dynamically discover its neighbors so we have to manually assign 

### Point-to-MultiPoint Network
- each connection will be treated as separate PP connection so there is no need to elect DR & BDR
- the default hello interval is 30 second.
- the dead interval is 4 times the hello interval so it's 120 second.

# Single Area Network
- in the past when processors were fairly slow the the recommendation for larger network that have more than 50 router should be divided into smaller areas. but these days processors are significantly faster and powerful there is no need to dived the network to areas.
- one scenario that you might want to consider divide the network to areas is where you have special devices that can run OSPF, those device can be put together in their own area so they don't have to deal with a lot of LSA, and this case is rare but other than that we usually want all routers to be in a single area.

## Configuring Single Area OSPF
```
(config)# router ospf "Process ID": PID is just an integer
-per network and interface basis
(config-router)# network "IP" "WildCard Mask" "Area"  
(config-router)# passive-interface "f0/0" : dont send hellos
(config-router)# auto-cost reference-bandwidth "1000 Mb/s"
```

All networks and interfaces in one command: ` (config-router)# network 0.0.0.0 255.255.255.255 <Area>`




## Router on stick
if we have traditional plain layer two switch that is not capable of interVlam routing, and we have multiple VLANs on that switch we need to connect to a physical router to do the routing between those VLANs, we connect to the router using a Trunk link, that carries traffics for all VLANs.
- on the router you go to subinterface
(config)# int "g0/0.1" : subinterface must be specified for each VLAN
(config-subif)# encapsulation dot1q "VLAN ID"
(config-subif)# ip address "IP" "MASK"

## Multilayer Switch (Layer Three Interface)
## SVI Switch Virtual Interface
On a multilayer switch each VLAN could have a virtual interface that encompasses all ports on that VLAN associated to it, and can be assigned an IP Address just like the router's interface. and that SVI routes traffic for all vlan ports
- Creating SVI:
```
(config)# int vlan "vlan id"
(config-if)# ip address "IP" "mask": like u normally would do with the router's interface.
```
- To be able to route packets, ip routing must be enabled.
(config)# ip routing
- To connect the Multilayer Switch to a router, we need ROUTED PORT, the port on the Multilayer Switch connected to the router need to be routed port, it needs an ip address, and by default all the switch's ports are in switchport mode, to change that, use "no switchport".
```
(config-if)# no switchport
(config-if)# ip address "IP" "MASK":  just when connecting two router, we can use /30 mask.
```