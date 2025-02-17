#EIGRP Concepts
#Cisco #Networking #IT

EIGRP is mainly Distance Vector RP, but has some link-state protocols characteristics, that's why EIGRP is sometimes referred to as a "hybrid routing protocol" or an "advanced distance-vector protocol". For the hybrid part because, it synchronizes network topology information between neighbors at startup like plain like DV RP, and then sends specific updates only when topology changes occur (bounded updates) just like link-state.

It sends traditional distance-vector updates that include information about networks plus the cost of reaching them from the perspective of the advertising router.
>Note: EIGRP has a default hop count of 100, with a maximum of 255. But this is used differently than RIP, it doesn't define whether the route is unreachable or not after the maximum hop count is reached, it only define the maximum hops that an EIGPR packet can traverse, just like TTL field. Practically limiting the size of any EIGRP AS network to be less 255 hop. !Note: you won't have a network with 255 hop anyway, even if you do you must do something about the TTL.

# EIGRP facts:
Uses hello messages with the multicast address of 224.0.0.10 to discover EIGRP neighbors.
The default hello timer is 5 sec, Hold down timer is 3 times the hello (15 sec).
Timers don't have to match between neighbors
The Ks and AS must match between all nodes in the topology not just between neighbors

# Exchanging routing information:
-> Becoming neighbors: for two EIGRP routers to exchnage routing information they must first for nieghborship, and for that the following parameters need to match:
• AS numbers
• K values
!Note: the timers don't have to match in order for the two device to become neighbor and exchange routing information.
After two devices see each others and agree to become neighbors, the start by exchanging their full RIB, which is done in unicast. So the multicast is only used for finding the neighbors but the actual exchange of the routing information is done as unicast.
!Note: If the route is not installed in you RIB you can't advertise it.
Just like OSPF, EIGRP uses its own transport protocol to carry routing information called RTP (Reliable Transport Protocol) protocol number 88, which allows for both reliability as the name implies by using Ack and sequence number to put packets in the correct order if they got reordered.


# Queue count: 
Because EIGRP uses multicast for the discovery and unicast for the actual exchange, it may be the case that there is an issue with the unicast, but multicast is okay, so the neighbors can't converge.
If the queue count is 0, this means that the neighbor formed neighborship and they converged. 
show ip eigpr neighbor !To display the queue count. 

After the first full exchange only changes in the RIB will be advertised.

Passive routes:
when you issue "show ip eigrp topology" each route will be prefixed with character code:
P: Passive: The route has converged and can be used for routing.
A: Active: The route is through a re-converged process, like the case when the link goes down or the neighbor disappear so the router will send queries to other routers, this means that the DUAL algorithm is currently doing its job.

