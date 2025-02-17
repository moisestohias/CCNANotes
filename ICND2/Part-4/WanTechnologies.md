WAN technologies:
#Cisco #Networking #IT

>Note: MPLS is in its own document.

- Metro Ethernet, is a technology that defines how to use Ethernet links between a customer site and the SP
- Metro Ethernet(Metropolitan): is a man topology that based on ethernet typically uses fiber optics.
the Advantages of metro ethernet over some of its competitor like SONET is it can be easily attached to a LAN and is less expancive. 

-SONET (Synchronus Optical Network) : a fiber optic multiplexing standard that allows multiple channels of comunication to share a fiber optic cable. 

- VSAT: Very Small Aperture Terminal : a WAN technology that uses a small satellite dishs installed at the network location for two-way communication via satellite. used for location that cannot have a wired internet connection . the data rate is usually between 56Kbps and 4Mbps.
- Very Small Aperture: refer to the size of the dishs which typically less than three meters.
- latency due to that signal travel slower in the air .
- VSAT is to whether condition.

- T1,E1,T3,E3 circuit: are digital circuits, and they have clocking and are syncronized they know when a bit start and when a it stop. 
- T1 circuit was originally desiged for telephone networks, each circuit is devided into 24 channels each channel called DS0.

- Because that T1 is an old technology. each of those DS0 has 64Kbits bandwidth which come from the fact that most of the human voice falls under the 4K Hz. and to replicate the same wave form to analog again Nyquist Theorem sugests that we use at least duble the maximum frequency of the original signal as sampling rate. the result is, to carry the voice data for one channel we use 8KHz as the sampling rate. the bit dipth is 8bit. 8K*8=64K.
- the bandwidth of T1 circuit:  
T1 Frame = 24channels * 8bit * 1 farmig bit = 193 bit
T1 bandwidth = 193 bit * 8K = 1.544 Mbps
- Because we never ever send a singal T1 frame by it self, we send either 12 Frames aka Super Frame (SF). or 24 frames aka Extended Super Frame (ESF).
> Regardless of whcich circuit we are using the circuit is always terminated on a digital modem either CSU or DSU Channel/Data Service Unit.
CSU/DSU Channel/Data Service Unit: is a layer one device that acts as a digital modem and connect to a digital circuit e.g T1,E1,T3,E3.
- we also use a layer two protocol like PPP.

Circuit   :   T1  |  E1   |  T3   |  E3 
Bandwidth : 1.544 | 2.048 | 44.7  | 34.4

Integrated Service Degital Network (ISDN):
digital circuit technology, where a circuit is devided into a logical B channels (which carry voice, data), and D channel (which carry signalling protocols). 
BRI BASIC RATE INTERFACE: an ISDN circuit that contain 2 64Kbps B channels and one 16Kbps D channel. 

