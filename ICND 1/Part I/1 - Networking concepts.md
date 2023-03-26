## 1 - Networking concepts:
# Networking Devices: 
+ Collision domain: Is a network segment where only one device can talk at a time. if two or more devices transmitted a signal at the same instant, the electrical signal collides and becomes garbled.
+ HUB (a.k.a bits spiter, not used today): Is a layer one device that connects net devices in a single collision domain.
-When an electrical signal comes in one hub port, the hub repeats that electrical signal out all other ports (except the incoming port). By doing so, the data reaches all of the rest nodes connected to the hub. in another word the hub floods each frame out all other ports (except the incoming port).

+ MAU (Media Access Unit): Used in Token Ring Net (90s tech) to physically interconnect Net devices (not used today). Physically the devices will be connected in a star fashion, but logically they are connected in ring, since the token circulates between devices carrying the information and only flow in one logical direction. Similar to a Hub MAU doesn't make a forwarding decision, it only forward token between devices.
The pedantic definition: A Media Access Unit (MAU) is a device used in computer networks to control access to shared media. It is responsible for granting access to the media to devices on the network. It is usually a part of a network switch, but can also be a stand-alone device. The MAU works by monitoring the network for packet requests and then granting access to the media to the device that requested it. It also ensures that only one device can access the media at a time, preventing collisions and data corruption.
!Note:Because the transmission is deterministic in MAU, only the device possessing the token can write, there is no concept of collision domain.
+ Bridge: is layer two devices that forward Ethernet frames based on the MAC address, it is the predecessor of modern switches, it separates the network into two collision domains, unlike switches it used software. 
+ Switch: is layer two device that forwards Ethernet frames based on the MAC address, each port of the switch is a separate collision domain, uses ASIC (App-Specific Integrated Circuit), that's why the switch is so fast.
+ Router : is layer three device that makes forwarding decisions based on the Routing table.

+ NIC (Network Interface Card): is a computer card that connects the host to the network. Also called LAN card, provides a physical interface between the host network, and the network medium.

#  Transmission Media:
+ Twisted Pairs:
- the medium consists of 4 pairs of conductors twisted against each others to protect from EMI(Electro Magnetic Interference).
- Exist in two main families: Shielded & unshielded
- it's the common medium type, although FiberOptic is becoming dominant even in the data center  .

* A Star Configuration
In the 1980s, 10Base-T began to be widely deployed. Using inexpensive and flexible twisted pair "telephone wires," 10Base-T Ethernet ("T" means twisted) was easier to install than previous "thick" and "thin" Ethernets that used coaxial cable

* cat3 > 10BASE-T (T: twisted Pair, not coaxial like 10BASE-2 or -5), and PBX Private Base eXchange, (telephone lines that can be used for Ethernet).  both are old, and was the first medium used.
* cat5  > 100BASE-T
* cat5e > 1000BASE-T
* cat6  > 1000BASE-T
* cat6a > 10GBASE-T  a:(Augmented)

+ 100Base-T, 100Base-T4, 100Base-TX and 100BASE-FX
100Base-T uses two pairs of wires in Category 5 UTP cable, 
100Base-TX requires two pairs in Category 6 cable.(h-quality t-pair)
100Base-T4 uses all four wire pairs in older Category 3 cables
100BASE-FX: fiber optic cables

+ Straight-Through and Crossover 
There are two cable set-ups Straight-Through and Crossover Cable, where the Straight was used to connect different devices (switch-PC), and Cross used to connect similar devices (Switch-Switch),  
g: green & white
G: green

gGoBbObB: T-568A
oOgBbGrR: T-568B
T-568A <-> T-568B : cross over cable
T-568A <-> T-568A, T-568B <-> T-568B : straight through cable
!Note: wire 1 and 2 are used for sending, 3 and 6 are used fro receiving.

!Note: Most modern devices port's support MDIX (Media Dependent Interface Crossover) as suppose to MDI (Media Dependent Interface). MDIX can determines which pins will be used to send and which are going to send.

# Fiber Optics:
- Since they use light pulses instead of electricity to send and receive data this obviates the need to protect against EMI.
- used for high speed and long distances.
### There are two types of FO:
- Multi-Mode FO  (MMF): used for short distances and inside organization. 
- Single Mode FO (SMF): used for long distances, and high bandwidth.

### FO cable structure: 
-Each cable has two strands, one for sending and one for receiving, 
-Each strand composed of core(the inner part) and cladding(the outer part), the refracting index between the core and the cladding is the difference and this cause the light to reflect inside the core to travel to the other end of the cable **

### Network Diagrams:
!Note: Network topology (diagrams): is a depiction of the network on paper or a screen
!Note: In networking diagrams, a cloud represents a part of a network whose details are not important to the purpose of the diagram. 

carrier sense multiple access with collision detection.

### RJ-11
RJ: stands for Register Jack, used to interconnect telephone equipment, or to connect to DSL modem.
has a hinged locking tab to lock itself in place
resemble the RJ-45 

### F-type connector: is the coaxial cable male jack
A screw on connector
