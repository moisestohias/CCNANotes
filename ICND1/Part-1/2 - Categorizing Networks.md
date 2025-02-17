## 2 - Categorizing Networks
We can categorize networks based on different characteristics of the network (Geography, Architecture, Topology):

# We can categorize networks based on its:
## Geography: How widely disperse are the network components?
+ LAN (Local): within a single building, couple hundreds meter (high-speed, locally centralized).
+ WAN (Wide): sites are geographically dispersed, relatively slower than LAN, sites connect into ServProv)
>Note: When drawihng WAN topology diagram, the connection between the sites and the SP is usually drown like a continuous lightning bolt to indicate a serial connection, other times is drown as a dashed lightning bolt to indicate that the circuit is not always up, is only brought up on demand like a SDN-BRI or dial-up model. Both of these are technology that have been used in the past, and they're not used anymore.
+ MAN (Metropolitan): not implement that much, high speed,
+ PAN (Personnel): limited number of devices withing a limited distance and throughput,
+ CAN (Campus): high speed, interconnect Nearby Buildings, used in Universities and  big companies with many facilities.

## Topology : How it's connected: Start, Ring ,Bus
>Note: Topology: is the layout of how devices are connected in a network.
+ **Physical Topology**: describe how devices physically connect.
+ **Logical Topology**: describes how the Traffic flow in a network.

+ Ring : connect devices in circular fashion, Token Ring used to be Ring topology with speed of 4Mb/s or 16 4Mb/s.
>Note: FDDI (Fiber Distributed Data Interface) used a ring topology, and fiber optic cabling to support speed up to 100Mb/s
+ Bus : Old, commonly used in early Ethernet net, used whats known as backbone that all devices are connected to, which is 50 ohm coaxial cable unlike 75 ohm used for TV and shit.

There are mainly two types of cable used in this setup:
+ THINNET(10BASE2):
	10 : 10Mbps, BASE:BASEBAND,
	2 : distance limitation of 185 ~ 200.
+ THIKET(10BASE5)
	5 : distance limitation of 500 m

- Start TOPOLOGY: is the most used topology, if one link fail other devices continue to function, but if the central device fail the whole network will fail, this is know as a single point of failure.

- FULL MESH TOPOLOGY: every site (or device in net) connect to every other site, if one link goes down every site (device) will continue to connect properly to all other devices bc of the redundant links.
Number of links = n*(n-1)/2 ; n=#of devices(sites)

- PARTIALLY MESHED TOPOLOGY: hybrid HUB & SPOKE and FULL MESHED topology , where redundant links are only added where needed.

- FDDI (Fiber Distributed Data Interface):
used a ring topology and Fiber Optic Cabling to support speed of 100 Mbps.

- HUB & SPOKE TOPOLOGY: remote site connect back to central site (spoke) if one link goes down the. other site will not be able to connect to that site who lost its link.

## Architecture : where the client get their resources from.
+ CLIENT SERVER ARCH: dedicated server that other devices can access or request from.
+ PEER TO PEER : client can serve each others.
