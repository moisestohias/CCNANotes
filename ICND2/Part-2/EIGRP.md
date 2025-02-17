# EIGP 
#STP #SpanningTreeProtocol #Cisco #Networking #IT

>Note: In EIGRP when you look at the output of `show ip eigrp IP/Mask` you should see the state of the route state as **passive**, meaning the route has converged and can be used.

# Always before thinking about configuring any routing, first:
1. **Make sure you have the correct IP addressing scheme in place(crucial)**
   - **Subnetting:** Confirm that subnets are correctly defined and there are no overlapping IP addresses.
   - **IP Addresses:** Ensure that each device has a unique IP address within its subnet and that these addresses are properly assigned.
2. **Check the reachability between the neighbors**: (ensure devices can communicate with each other)
   - **Ping Tests:** Use the `ping` command to test connectivity between neighboring routers or devices to ensure they can reach each other.
   - **Routing Table Verification:** Make sure that routes to the neighboring devices are present and correct in the routing tables.
> Note: These steps are essential to avoid headache.

EIGRP is a distance vector protocols, Unlike link-state protocols it doesn't have full view of the topology, which is a good thing, since it allows flexible deicing (you can filter/summarize anywhere in the network)

>Warning: If you are using EIGRP in your network you have to use one mode for the entire network, or at least make the the routers that uses the same mode contiguous, other wise you will end up in unpredictable result **due inconsistent use of the metric**.

Distance Vector Routing Protocols Golden Rule 🏆: **if the route is not installed by DV RP process in your global RIB you can't advertise it**. 
>Note: After you enable EIGRP on the devices, they check with each others what mode they are running, if both peers are running in the named mode they use the wide metric. Otherwise the named peer will fall back to the classic metric.

>Note: Any change that you do in EIGRP will drop the neighborship and reform again, so most of the change that affect EIGRP won't be hit-less.

# EIGRP three tables (Interface, Neighbors, Topology):
These are the table that are maintained and used by EIGRP
+ **Interface Table**: EIGRP keeps track of EIGRP enabled interfaces: `show ip eigrp interfaces`
+ **Neighbor list (Table)**: EIGRP keeps track of other EIGRP neighbors by forming neighborship with them, you can display then with: `show ip eigrp neighbor`
+ **Topology Table (RIB)**: After OSPF runs the SPF on the LSDB and finds the Shortest Path Tree (SPT) to put them in the rib, so you can check with `sho ip ospf rib`, EIGP also keeps track of the topology in the its topology table, which you can display using `show ip eigrp topology`

# Timers:
+ Hello(5s): Sent every HellorInterval dicscover neighbors and maintaining the neighborship
+ Hold(15s): How much you should wait on me before you declare me dead after not hearing hello from me.
>Note: In **EIGRP Hold interval** is the router telling his neighbor how long to wait on it before you declare it dead, and it doesn't have to match between neighbors. In the case of **OSPF the Dead interval** must match between neighbors, and its basically an instruction (by you admin) to the router on how much to wait on neighbors before declare them dead.

# Traffic:
When you configure EIGRP on the router, it start sending Hello every HellorInterval to the **multi-cast address 224.0.0.10** (with his IP as source), to dicscover neighbors, when neighbor send his Hello, the updates between the neighbors is unicast all subsequent updates, but the **Hello is always Muli-cast**.


# EIGRP Classic Mode Metric calculation: (Warning: It's really fanky)
Mnemonic: BDRLM: Big Dogs Really Like Me
EIGRP uses a composite metric (cost), consisting of multiple link attributes, to calculate the come up with the final cost value. EIGRP cost can consider multiple link attributes: bandwidth, delay, load, reliability and MTU. 
>Warning: By default only **bandwidth** and **delay** are used in actual calculation, due to their dynamic and potentially fluctuating nature. If you do include them you basically running these calculation every two minutes (or seconds not sure) which doesn't make since to put this load on the control plain. **MTU** is not included at all in the metric calculation, it's only meant to be used as a tie breaker (look below for detail).
+ **Bandwidth**: EIGRP only considers **the lowest bandwidth link** in its metric calculate (the bottleneck) Reasonable! specifically is calculated by dividing 10,000,000 `(10**7)` by the speed of the slowest link (in Kbit) along the path. 
You can specify the bandwidth of the interface using the `bandwidth` interface command which is in Kbit/s, otherwise the speed will be inferred from the in physical operational speed of the interface.
>Note: changing the bandwidth of t he interface only affect the calculation of the application that use this value, no the actual interface's speed used.
+ **Delay**: is accumulative, every time we exit an interface we add the delay in "MICROSECOND" of exiting that interface -which can be seen in the output of `show int <int-Id>` next to the speed and divide the total by 10. 
You can also specify the delay of the interface using the `delay` interface command, which take any value and scale it by 10, for example if you configure "delay 10" this will be 100 Microsecond, otherwise the delay will be inferred from the physical interface.
With Classic Metrics, EIGRP is capable of describing the delay in the range of 10 to 167,772,140 microseconds (1 to 16,777,214 tens of microseconds when configured manually).
>Note: A delay of 16,777,215 tens of microseconds is used to indicate an infinite distance and is the key to the ability of advertising an unreachable network. Split Horizon with Poisoned Reverse, Route Poisoning, withdrawing a route, all these techniques in EIGRP use the maximum delay as an indication of an unreachable route.
>Note: some documents refers to the delay the hop count, which is effectively true in the case of all links in the network have the same speed and delay.
+ **Reliability**: is reported based on the ratio between the number of successfully received and the number of all received frames. Take the lowest reliability along the path.
+ **Reliability**: is reported based on keepalives sent between the routers, if the keepalives are dropped, reliability decreases. Take the lowest reliability along the path.
+ **Load**: is based on link utilization. Take the highest load along the path.
>Note: It's recommended to only use bandwidth and delay, since reliability and load can change often thus introducing instability.
>Note: It's recommended to not include the load and the reliability in the metric calculation because these values change often, thus causing re-convergence.
+ **MTU**: **Originally intended to be used as a tie breaker**, if you have two links with the same metrics, you can use the MTU as a tie breaker, but generally the **MTU is rarely changed and it's not recommended that you change it**.
>Note: delay is summed, bandwidth and reliability are minimized, load is maximized.
Bandwidth = 10,000,000/slowest-link-speed
Delay = total-exit-delay/10
`metric = [(K1*BW) + (K2*BW)/(256 – Load) + (K3*Delay)*(K5/(Reliability + K4))] * 256`
>Note: the term (K5/(Reliability + K4)) is only considered as part of the formula if the K5 is not 0.
>Note: The Ks are factor that we can change to influence the metric calculation, by default K1 and K3 are 1 and the rest are 0. So by default the metric is: `(Bandwidth+Delay)*256`
`Bandwidth = 10,000,000 / slowest-link-bandwidth` in Kbit
Delay = sum of the output delay of each outgoing interface along the path / 10
>Note: Decimal results are rounded off
>Note: By default EIGRP does not load balance. 

>Note: The bandwidth and delay metric components are carried in EIGRP packets in their scaled form. This requires each router to descale them first to obtain the Bandwidth Min and Delay Summed values to perform the necessary minimization of bandwidth and summing the delay, and then scale them again when computing the composite metric and advertising the route to its neighbors.

## A different way to look at the metric formula:
>Note: This is how Brian McGahan showed it, which the form used in the RFC.
K1=1, K2=0, K3=1, K4=0, K5=0
`metric = [(K1*BW) + (K2*BW)/(256 – Load) + (K3*Delay)] `
where: 
`BW = 10**7/slowest_linkBW * 256`
Delay = Sum/10 * 256

## Traffic Engineering with Metric:
>Note: Because the bandwidth in not accumulative in EIGRP metric calculation, if you have a network where all the links have the same bandwidth, the bandwidth won't even come to play, effectively you are only using the accumulative delay.
>Note: like all IGPs, you can't change the metric per prefix or per destination, but in EIGRP you can offset the final metric for a specific destination practically changing the metric for that prefix !more on that later!.
>Warning: since the Ks need to match on all nodes, if there is a question that tells you to only use the delay for the metric calculation, you have to change the ks for all nodes on the entire network.
>Warning: if you want to change the metric of EIGRP, by changing the link attributes like the bandwidth this will affect other applications, namely QoS. 
>Warning: if you want to change the metric of EIGRP, by changing non-accumulative attributes like the bandwidth (load and reliability if used) this will affect the calculation for other nodes that uses the link. 
!TIP: if you want to change the metric for one link, it's easier to only consider the delay (accumulative attribute) rather then non-accumulative attributes.

# Adjusting EIGRP Metric:
there are tow way to change the EIGRP metric: 
## Adjusting the Ks value (EIGRP's formula factors):
The Ks factors need to match on all routers.
## Adjusting the link attributes:
To change the EIGRP metric you can change .

# EIGRP Named Mode Metric calculation:
Once we go above 10G, EIGRP can't distinguish between speed of the link any more, when using the classic mode. Like the case with OSPF, when they implemented the protocol, they didn't though that we will have that higher speed links. From the EIGRP classic mode, 10G, 20G, 40G, 100G ... they all the same in them of bandwidth as well as delay, where the bandwidth is 256 and the delay is 10 microsecond. To account for that the wide metric is used, where the size of the variable is now 8 Byte (64 bit), as suppose to 4 Byte like the case in the classic mode.
The calculation is still the same, with one minor changes: mainly the whole thing is now scaled by 65535 instead of 256. 
The Bandwidth is now called the Throughput. 
The Delay is now called the Latency, and it's calculated by PicoSecond
Effectively, EIGRP named mode metric = `(Bandwidth+Delay)*65535`
+ K6: by default is 0, this factor is added in the new amendment along with wide metric, to account for future attribute that they may include in the EIGRP metric calculation, namely the jitter and energy (energy drown by the SFP module to push information into the fiber). 
+ RIB scale: because the wide metric is now 64 bit, the value can get quit large than what can be fitted into the cost field which 4 Bytes (32 bit) field value that installed installed in the global routing table, the amendment introduced new value called "RIB scale" which basically takes the feasible distance and divide it by this value which is by default 128. To change the RIB scale value to other value than the default, under the af mode:
(config-router-af)# metric rib-scale <value>


# Terminologies
EIGRP named mode (aka multi-af mode)
+ **Reported Distance and Feasible Distance (Metric Calculation)**
The **Reported Distance** is the metric information (cost) the advertising router sends, when the receiving router receive that information it adds its receiving interface delay to the advertised delay and compare the bandwidth of its receiving interface to the advertised one if its lower it substitute is bandwidth. Then use those values to calculate the final cost, which is called the **Feasible Distance**. 
To display both the reported and feasible distance for each route, use: `show ip eigrp toplogy`  !The values between parenthesis are the feasible/reported distance.

+ **Successor route**
The primary route to a subnet, based on the route having the lowest Feasible Distance (FD) of all route in the EIGRP topology.

+ **The feasibility condition**
The feasibility condition is a selection process used by EIGRP to **identify potential backup routes**, known as **feasible successors**, for a primary route (the successor). For a route to be eligible to become a feasible successor, it must satisfy the feasibility condition, which states that *the reported distance of the potential successor route must be less than the current router's feasible distance of the successor route*. This condition aims to prevent the introduction of false routing information into the network and ensures that the feasible successor is a genuinely viable alternative to the primary path [see example](youtu.be/e5qYqNX6f0k?t=1824). 


+ **Successor route**
After the Feasible Distance for a route is calculated and there is not better route (with lower metric) that route will be installed in the RIB and will be called the successor route.
If there are multiple routes to the same prefix but with different feasible distance the one with lowest metric (distance) win, that is the successor, and the other are stored as a backup in case of failure, and they are called **Feasible Successor routes**. 
To display the list of successor and feasible successors route for each each prefix, use:
show ip eigrp toplogy !The first route is successor

+ **DUAL (Diffusing Update Algorithm)**
Ikf the successor route fail and no backup (feasible successor) route is found, the route will send query to its neighbors asking for routing information the lost prefix,   


# Configuring EIGRP
### EIGRP Configuration modes (EIGRP modes)
There are two modes for configuring EIGRP: 
## Classic mode : 
+ Prior to IOS 15
+ Uses AS number for creating the process. 
+ The configuration syntax is not consistent and fragmented. 
For instance configuring authentication, the syntax is under "ip authentication mode eigrp <AS-number>", or "ip summary-address eigrp <AS-number>" or for doing summarization.
## Named mode: (aka Multi-AF mode (AF: Address-Family)):
+ IOS 15 and later.
+ Uses name (string) instead of AS number for creating process. 
+ Enhanced and organized the configuration syntax to provide better configuration hierarchy, 
+ Unified the configuration for both stacks IPv4 and IPv6.
+ Also adds additional features: wide-metric, support for IPv6 VRF...
>Note: behind the scene the both operate the same way, so its a merely a configuration syntax enhancement.  

There are two ways for telling which EIGRP mode you are running, either by displaying the running-configuration, or when you issue "show ip eigrp topology <prefix/mask>" the "Total Delay" will be measured in picoseconds as supposed to microsecond, this means you are using wide-metric, which is only available in the named mode.

>Note: in the output of "sho ip protocol"
Routing information sources: This section lists all neighbors that we are learning routing information from them (gateways).

## EIGRP Classic Configuration Modes:
router eigrp 1 !Create IPv4 EIGRP process using the classic mode.
ipv6 router eigrp 1 !Create IPv6 EIGRP process using the classic mode.

+ The network command in EIGRP classic mode: In EIGRP you can choose to either specify the wildcard mask or not.
if you don't specify the mask, EIGRP will use the default mask based on the class.
 network 10.0.0.1 0.0.0.3   !Enable EIGRP only on those interfaces whose IP fall in subnet /30.
 network 10.0.0.193 0.0.0.0 !Enable EIGRP only on the interface with this exact IP
 network 172.16.0.0         !Enable EIGRP on all sunbet of 172.16.0.0/16

+ EIGRP RID classic mode:
The RID selection process for EIGRP is exactly the same as in OSPF. 
To configure the RID in EIGRP however, you use: 
- IPv4 RID: "eigrp router-id <RID>"
- IPv6 RID: "ipv6 eigrp router-id <RID>" >Note: IPv6 RID has to be IPv4 address

+ EIGRP timers configuration:
The timers are interface dependent attributes, and they don't have to match with peen neighbor. The The default hello timer is 5 sec, Hold down timer is 3 times the hello (15 sec).
>Note: The dead timer is called hold-down in Cisco world this goes for EIGRP as well as CDP
You can only display the hello timers on the interface:
show ip eigrp interface detail !The hold down you have to infer it.
### To configure the timer you go to the Interface configuration mode:
>Note: you can configure both hello and hold-down timer independently. If you configure only the hello, the hold-down will be automatically adjusted to be 3 times the hello. But when you can configure the hold-down to be any value, not specifically 3 times the hello.
ip hello-interval eigrp <AS-number> <Hello-timer>
ip hold-time eigrp <AS-number> <Hello-timer>


+ EIGRP Authentication (classic mode):
EIGRP only supports MD5 authentication. To configure authentication, you only enable it at the interface level, not at the global EIGRP process level.
ip authentication mode eigrp 1 
ip authentication key-chain eigrp 1 MOISES
>Note: The syntax not hierarchical (not intuitive). Why not start with "eigrp 1" then "authentication"

## EIGRP Named Configuration mode (EIGRP Named mode):
In this configuration mode the EIGRP process looks unrelated to the IP version. 
You start by creating the process and using a string, which doesn't have to match with the other peers. Then you specify the IP version, and the AS number that needs to match with other peers. 
>Note: The AS number for IPv4 and IPv6, can be different or the same doesn't matter as long as it match with the other peers.
>Note: the RID, is per AF,

+ EIGRP RID in named mode:
To configure the RID in EIGRP named mode, go under the AF mode: 
- IPv4 RID: "eigrp router-id <RID>"
- IPv6 RID: "eigrp router-id <RID>" 
>Note: The RID can be the same or different for IPv4 & IPv6, doesn't matter as long as it matches with the other peers. But IPv6 RID has to be IPv4 address.

+ The network command in EIGRP named mode, goes under the AF sum-mode, and works exactly the same.
router eigrp MOISES
 address-family ipv4 autonomous-system <AS number> !Set the AS number.
  network 0.0.0.0
 address-family ipv6 autonomous-system <AS number> !Set the AS number.

+ EIGRP interface commands: for EIGRP interface level commands, like authentication and timers, you now configure them under the "af-interface <interface family> <int-ID>" EIGRP configuration mode, which make since, you don't have to keep jump back in forth between EIGRP and interface configuration modes.
+ af-interface default: this mode allows you to configure the default EIGRP configuration for all interface, in another word all attributes configured here will be inherited by the EIGRP enabled interface, unless you have specific configuration on the interface. For instance you can apply you authentication here, and all EIGRP interfaces will inherit those attributes.

## MTR (Multiple Topology Routing):
Is a Cisco IOS feature that allows you to configure different interface to be in different logical topology, and have different routing table for each topology where the forwarding process (data plane) is separate.
Complete guide: cisco.com/c/en/us/td/docs/ios/12_2sr/12_2srb/feature/guide/srmtrdoc.html#wp1055753
>Note: is not commonly used.
>Note: I am mentioning MTR here, because when you look at the EIGRP named configuration you will see "topology base" which is the base topology in another word the global routing table.
+ Changing process specific attributes: to change global EIGRP process specific attributes, things like AD, maximum-paths for load distribution, default redistribution metric, etc, you configure it under the base topology sub mode.

## Difference between EIGRP classic and named mode:
From operation point of view, they are exactly the same, thus interoperable. 
The named mode does support additional feature however, like mentioned before, namely wide-metric to account for high speed links like 100G and 400G, 
The metric field in the classic mode implementation is smaller than to store high  

## Upgrading the configuration mode:
The configuration upgrade is automated as for IOS 15.4S: (config-router)# eigrp upgrade-cli <name>
The upgrade is hitless: no need to schedule a maintenance window for this. The data plane won't be interrupted. Permitting graceful restart, where the routers will tell each other that the neighborship will flap, but don't flush the RIB.

::Graceful Restart (aka NSF Non-Stop-Forwarding): is a common operation among higher end chassis based devices. Where, if the primary supervisor engine fail, the standby supervisor kicks-in automatically, and tells the other neighbors that everything is fine, you can still forward your traffic to me, don't interrupt the data plane, but the control plane needs to refresh.


# EIGRP verification:
Regardless of the configuration mode the show commands are exactly the same
show ip eigrp topology 
!displays the successors and feasible successor routes.
>Note: this is practically the EIGRP RIB.
show ip eigrp topology <prefix/mask>
!displays information about the route: successor and feasible successors, FD...
>Note: in the case of named mode, the FD is not the metric that will be installed in RIB, but the metric that will be installed in the RIB, will be just next to it.
show ip eigrp topology all-links 
!displays the entire EIGRP known routes.
show ip eigrp topology summary
!displays the AS number and RID, number of EIGRP routes, number of local EIGRP interface and the number of EIGRP neighbors on that interface.
show ip eigrp interface [detail | brief]
!displays the local EIGRP interfaces and the number of EIGRP neighbors on those interfaces.
>Note: passive interfaces won't show up in this output.

# Interpreting output of show commands:
+ `show ip eigrp neighbor`
  + **Q cnt**: queue count, if this value is not 0, this means that two neighbor haven't converged, and there may be a problem with unicast.
  + **SRRT**: Smooth Round Trip Time, if this value is high for whatever reason, this maybe problem, like layer one transmission problem.
  + **SeqNum** similar to the TCP sequence number, since EIGRP uses its own transport proto
+ `show ip protocols eigrp`
  `Routing Protocol is "eigrp 1"`: the router is running EIGRP with an AS number of 1.
  + **Outgoing/Incoming update filter (distribute-list)**
  These lines indicate if there is any filtering going on restricting which routes can be sent out or received in EIGRP updates. 
  `Outgoing update filter list for all interfaces is not set`
  `Incoming update filter list for all interfaces is not set`
  >Noe: In the case there is no update filter lists configured.

  + **Wheter or not default-routes can be accepted or not**
  ```sh
  Default networks flagged in outgoing updates
  Default networks accepted from incoming updates
  ```
  These lines indicate whether or not default routes are being advertised in outgoing EIGRP updates and accepted from incoming EIGRP updates.

  `EIGRP-IPv4 VR(B) Address-Family Protocol for AS(1)`
  This section provides details about the EIGRP configuration and operation for the IPv4 address family in the specified AS.

  + `Metric weight K1=1, K2=0, K3=1, K4=0, K5=0 K6=0` metric weights
  + `Metric rib-scale 128` metric scaling factor (wide in this case)
  + `Metric version 64bit` metric precision

  + `Soft SIA disabled`: This indicates that the "Soft State Stuck-In-Active" (SIA) mechanism is disabled. SIA is a mechanism used to address potential routing instability issues when a query for a route goes unanswered for an extended period.

  + `NSF-aware route hold timer is 240`: This specifies the route hold timer for Non-Stop Forwarding (NSF) awareness. *NSF is a feature that aims to minimize the impact of a route processor switchover on network convergence.*

  + `Router-ID: 1.1.1.1`

  + `Topology : 0 (base)` Which topology subsequent lines (), These lines specify various params (limits and counts) related to EIGRP operation.
  >Note: Cisco support **Multi-topology-Routing (MTR)** that allows you to use multiple toplogy on same underlay.
    + `Active Timer: 3 min` :    The "Active Timer" indicates the time period for which a query remains active before considering a route unreachable. 
    + `Distance: internal 90 external 170`: AD used in this topology
    + `Maximum path: 4`: maximum number of parallel paths EIGRP can utilize for load balancing
    + `Maximum hopcount 100`: 
    + `Maximum metric variance 1`
    + `Total Prefix Count: 3`
    + `Total Redist Count: 0`

  `Automatic Summarization: disabled` whether Automatic Summarization is enabled or disabled

  ```
  Routing for Networks:
  0.0.0.0
  ```
  This section indicates the networks for which EIGRP is performing routing. In this case, the router is routing for the default route (0.0.0.0).

  ```
  Routing Information Sources:
  Gateway         Distance      Last Update
  10.0.0.2              90      00:00:05
  ```
  This section lists the routing information sources, along with their distances and the time of the last update. In this case, the router has received information from a gateway with the IP address 10.0.0.2.

  + `Distance: internal 90 external 170`: the AD for internal and external routes.

  Overall, the `show ip protocols` output provides an overview of the EIGRP configuration, topology information, metrics, and related settings on the Cisco router.


# Debugging EIGRP:
debug eigpr ?
address-family  EIGRP address-family debug commands
fsm             EIGRP Dual Finite State Machine events/actions
neighbors       EIGRP neighbors
nsf             EIGRP Non-Stop Forwarding events/actions
packets         EIGRP packets
service-family  EIGRP service-family debug commands
timers          EIGRP Timer Events
transmit        EIGRP transmission events

debug ip eigpr ?
<1-65535>      Autonomous System
A.B.C.D        IPv4 Address
neighbor       Neighbor debugging
notifications  Event notifications
summary        Summary Route Processing
vrf            Select a VPN Routing/Forwarding instance
<cr>



---

---

int e0/0
ip add 10.0.0.2 255.255.255.248 
no shu


router eigrp B
add ipv4 a 1 
net 0.0.0.0 


int loop 0
ip add 1.1.1.1 255.255.255.255
no shu
int loop 0
ip add 2.2.2.2 255.255.255.255
no shu
