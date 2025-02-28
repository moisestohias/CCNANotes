## 01- ETHERNET

How the switch works inside: It basically creates a circuit path between the communicating devices.

> Note: Cat 5, 5e, 6, 7 Ethernet all have the same distance limit of 100 meter.

ETHERNET was developed by XEROX in the 1972.
> Note: The IEEE 802.3 is a collection of IEEE standards, that define the Ethernet standard over varios speed, medium..

+ **10BASE5**: (also known as thick Ethernet or thicknet) was the first commercially available variant of Ethernet. The technology was standardized in 1982 as IEEE 802.3 which run at 10 Mbit/s (1.25 MB/s) over thick coax. 10BASE5 uses a thick and stiff coaxial cable up to 500 meters (1,600 ft) in length. Up to 100 stations can be connected to the cable using vampire taps and share a SINGLE COLLISION DOMAIN with 10 Mbit/s of bandwidth shared among them. The system is difficult to install and maintain. The devices were connected in bus topology fashion. 
10BASE5 was superseded by much cheaper and more convenient alternatives: first by 10BASE2 based on a thinner coaxial cable, and then, once Ethernet over twisted pair was developed, by 10BASE-T and its successors 100BASE-TX and 1000BASE-T. In 2003, the IEEE 802.3 working group deprecated 10BASE5 for new installations


+ **Vampire Taps**: is how devices attached to the coax cable in 10Base2/5 net, it has 15 pin adapter that called AUI that connect to the device.

| Ethernet standard | Media type             | Bandwidth-limit     | Distance limit |
| ------------------|------------------------|---------------------|--------------- |
| 10Base5           | coax(ThickNet)         | 10 Mbps             | 500m           |
| 10Base2           | coax(ThinNet)          | 10 Mbps             | 200m           |
| 10BaseT           | cat3 (or higher) UTP   | 10 Mbps             | 100m           |
| 100BaseTX         | cat5 (or higher) UTP   | 100 Mbps            | 100m           |
| 100BaseFX         | MMFO                   | 100 Mbps            | 2   km         |
| 1000BaseT         | Cat5e (or higher) UTP  | 1000 Mbps           | 100 km         |
... full table in 51.lesson 2 Ethernet distance.

> [!Info] Bridges were developed in the 1980, to limit collision domains, which allowed network to somehow scale, they used software to perform switching, so they were inherently slow,  and the processor used back then CISC/RISC (Complex/Reduced Instruction Set Computing).
> [!Info] Switches were introduced in the 90s, they uses ASICs (Application Specific Integrated Circuit), which is super fast (line speed). Another big advantage of switches is they have much larger number of ports.

> [!Note] switch (bridge) forwards unknown, multicast, broadcast frames.
> [!Note] ETHERNET only report on data integrity not on data recovery
> [!Note] Ethernet hub port can only receive or transmit at any one time, this Half Duplex mode uses CSMA/CD

If we want to use CSMA/CD requirement is to run in Half Duplex mode 

---

## Lesson 53 MAC address
MAC address: is a 6 bytes (48 bit) address, burned in to the NIC by the hardware manufacturer, usually written in Hex notation, each to Hex digits are separated by a colon or hyphen, sometime is written as four Hex digits separated by a dot. The 3 first bytes (24 bits) are known as the Organizational Unique Identifier (OUI) (aka vendor code), the other 3 bytes are device unique.

## Lesson 54 CAM/MAC address Table
Each switch keeps track of the known MAC addresses and the respective port, on a table known as CAM (Content Addressable Memory) table.

## Lesson 55 Collision Domain
Collision Domain: is network segment where only one device can talk at a time. 10Base5/10Base2 Ethernet networks are an example of a network with a single collision domain, since device are connected in a bus topology.
> Note: if an interface is configured to operate in a simplex model, it will use CSMA/CD (Carrier Sense Multiple Access With Collision Detection). The way CSMA/CD operate the interface will sense the carrier to see if the link if free to use (cost clear or not), if it's free, it will send its data. If a collision does occur the device sending will continue sending the whole frame, letting everybody on the segment that there was collision, this process known as Jamming. Then each device will set its random back-off time and try to send again.

> [!Note] **Jamming**: the continuation of transmission of a collided frame, in effort to let all device on the shared segment to detect the collision.

The Switches separate Collision Domains, letting every device connected to it to run on a full-duplex. Giving effective use of the bandwidth.

---

## Lesson 58 Frame Forwarding Options
+ Cut-Through Switching: supper fast, it starts forwarding the frame after examining the frame's destination MAC address. It was not used much because wasn't reliable since we can switch packet but at the end when we calculate the FCS and doesn't match the FCS of the frames so it's corrupted , bu today it's used in modern switches like CISCO 5000 Nexus Series.
+ Store-and-Forward : introduce small delay (latency) as the switch waits till the whole frame is received and check FCS of the frame with it's CRC to make sure that the frame is good, to be retransmitted.
+ Fragment-Free : Compromise between the Cut-Through and Store-and-Forward it wait to for first 64 Byte, and make sure they are ok and no collusion has occurred, so it conclude the frame is ok to retransmitted.
