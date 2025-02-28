## 3 - Protocol Reference Models:

## Standardization bodies:
+ ISO: International Standardization Organization
+ TIA/EIA: Telecommunications Industry Association/ Electronic Industry Association
+ IETF: Internet Engineering Task Force
+ IEEE: Institute Of Electrical And Electronics Engineers

## OSI
The Open Systems Interconnection model (OSI model) is a conceptual model that characterizes and standardizes the communication functions of a telecommunication or computing system without regard to its underlying internal structure and technology. 
Each layer provides a service to the layer above it. For instance the transport layer protocols provide services to the application layer protocols that reside one layer higher in the TCP/IP model. 


### Protocol Data Unit PDUs: (what name we give to the data at each layer):
+ Physical  layer : Bits
+ DataLink  layer : Frames, Packet
+ Network   layer : Packet, DataGrams
+ Transport layer : Segments, Packet 

# Responsibility of each layer of the OSI Model:

The Physical layer: defines the electrical, mechanical, procedural, and functional specification for activating, maintaining and deactivating the link.

## Physical:
  - Coding standard (How bits are represented through the medium:RZ,NRZ,AMI, Manchester..)
  - Wiring standard (wires and connectors,jack TIA/IEA 568b)
  - Physical topology (start,ring,bus)
  - Synchronizing bits (Async, Sync)
  - Bandwidth Utilization: depends on the used tech: Switch, WIFI or SDH/PDH.. (BroadBand (FM, AM, PM..), BaseBand (only one conversation at a time eg; Manchester..))
  - Multiplexing (TDM, Statistical-TDM, FDMA)
!Note: the Physical layer can be the most complex of them all.
## DataLink:
  + **MAC (Media Access Control)**: - Physical Addressing (MAC address 48 bits, 6 Bytes)
    - Logical topology (the signal path in the network, eg; MAU physically is a start topology but logically is a ring topology, Hub physically is a start but logically is a bus)
    - Transmission Method (Token Ring, CSMA/CD)
  + **LLC (Logical Link Control)**: - Connection services (only Implemented on special devices) (FLOW CONTROL: between devices in the LAN using the switch, ERROR CONTROl: re-sending the data if case of failure, Note: this feature only implemented in special devices)
    !Note: Using LLC is compulsory for all IEEE 802 networks with the exception of Ethernet.
    - Synchronization Transmissions:
      + Isochronous: both devices reference external clock and each have its turn (slot) to send data.
      + Synchronous: the sender and receiver exchange their clocks on a separate channel.
      + Asynchronous: each use its own clock but agree on start and stop bits.
    - Error checking : CRC (Cyclical Redundancy Check), Parity-bit

## Network:
  - logical addressing (IP), each device is assigned a unique IP to distinguish it from other devices.
  - Switching (Pack, Circuit, Message Switching (details are below)) How we send data from one device to another, *Note: we mainly use Packet Switching, hence the name of PDU of data in this layer.
  - Route discovery and selection (RP : OSPF,EIGRP,RIP)
  - Connection services (flow control, Packet Reordering -only used in the case of fragmentation-)

## Transport:
  - **Segmenting protocols (TCP/UDP)**: split data stream into small segments that can be exchanged in the network
  - **Multiplexing (Multiple Application and services use the same IP address)**
  - **Flow control**:
  - **Error Detection** (both TCP and UDP offer error detection)
  - TCP offers far more features than UDP:
    + **Reliability**: ensuring that data sent is received by the other end.
    + **Windowing** (Sliding window) allow the TCP windows size to grow based on the net reliability. (TCP windows size : the number of segment can be sent before an acknowledgment is received )
    > [!Note] Max Bytes per window (without getting an ACK) is 2_000_000 Bytes

  - **Buffering**: if ingress interface of the router is sending data to quickly than the  egress is able to output (like the case of LAN to WAN), the router will buffer the data in a queue.

## Session:
  - Setting up session
  - Maintaining session
  - Tearing down session
  !Note: The session layer is not extremely useful since the sessions are established by the layer 4 protcols. It maybe useful for logical segmentation of the functionality of certain applications, like VPN.

## Presentation:
  - Encoding: ASCII, EBCDIC (E Binary Code Decimal Information)
  - Encryption

## Application:
  - Services.

#  TerminoloJesus:
Asynchronous communication: means the transmitter and receiver do not share an external clock signal (such as would be transmitted over a "clock" pin or "clk+/clk-" pair on a cable). Where data can be transmitted intermittently rather than in a steady stream.

## DTE/DCE
- DTE (Consumer Side): Data terminal equipment. From a Layer 1 perspective, the DTE synchronizes its clock based on the clock sent by the DCE. From a packet-switching perspective, the DTE is the device outside the service provider’s network, typically a router.

- DCE (SP Side): Data communications equipment. From a physical layer perspective, CDE is the device providing the clocking on a WAN link, typically a CSU/DSU is the DCE. From a packet-switching perspective, the service provider’s switch, to which a router might connect, is considered the DCE.

## Ethernet is Asynchronous.
Asynchronous communication means the transmitter and receiver do not share an external clock signal (such as would be transmitted over a "clock" pin or "clk+/clk-" pair on a cable). Ethernet cables have no clock pins or pairs. Ethernet does not use a separate clock signal shared between the transmitter and receiver, so it is asynchronous.

Because Asynchronous communications buses don't share a separate clock signal, the transmitter has to encode each transmission in a way that allows the receiver to know when one bit ends and the next bit begins. Ethernet's solution for that is to start every transmission with a long series of alternating 0 and 1 bits -- the preamble -- which allows the receiver to temporarily synchronize its bit-clock with the transmitter's clock for the duration of that transmission. As soon as one frame ends and the next begins, the temporary synchronization must begin again.

## Data Link Lyaer
IEEE 802.2 is the original name of the ISO/IEC 8802-2 standard which defines logical link control (LLC) as the upper portion of the data link layer of the OSI Model.

The LLC is a software component that provides a uniform interface to the user of the data link service, usually the network layer. LLC offers three types of services (aka connection mode):
- Unacknowledged connectionless mode services (mandatory, except Ethernet)
- Connection mode services (optional)
- Acknowledged connectionless mode services (optional)
The LLC uses the services of the MAC sublayer, which is dependent on the specific transmission medium (Ethernet, Token Ring, FDDI, 802.11, etc.). USING LLC IS COMPULSORY FOR ALL IEEE 802 NETWORKS WITH THE EXCEPTION OF ETHERNET. It is also used in Fiber Distributed Data Interface (FDDI) which is not part of the IEEE 802 family.

The additional information added by the LLC sublayer is the LLC HEADER, which consist of DSAP (Destination Service Access Point), SSAP (Source Service Access Point) and the Control field.
The 8 or 16 bit HDLC-style Control field serves to distinguish communication mode, to specify a specific operation and to facilitate connection control and flow control (in connection mode) or acknowledgements (in acknowledged connectionless mode).

## Role of the LLC:
The two fields DSAP and SSAP (1 byte each) ALLOW MULTIPLEXING OF VARIOUS UPPER LAYER PROTOCOLS ABOVE LLC. However, many protocols use the SubNetwork Access Protocol (SNAP) extension which allows using EtherType values to specify the protocol being transported atop IEEE 802.2. It also allows vendors to define their own protocol value spaces.

+ SAP (Service Access Point) is an identifying label for network endpoints used in OSI networking.
The SAP is a conceptual location at which one OSI layer can request the services of another OSI layer.

## SNAP vs SAP:
+ SNAP (SubNetwork Access Protocol): Is a mechanism for multiplexing more protocols than can be distinguished by the 8-bit 802.2 Service Access Point (SAP) fields, on networks using IEEE 802.2

### Role of SNAP and LSAP:
The SNAP and LSAP fields are added to the packets at the transmitting node in order to allow the receiving node to pass each received frame to an appropriate device driver which understands given protocol.

# Network Layer : 
TCP/IP : is suit of layer 3 and layer 4 protocols, that uses IP at layer 3.

## Packet, Circuit, Message Switching:
+ Packet Switching: the data is divided into packets, a Packet contains a header with a source and destination address along with other info. This is what we use for network.
+ Circuit Switching : Temporary connection, the circuit is brought-up on as needed, similar to placing a phone call.   
+ Message Switching :  the data is divided into messages which are not necessarily delivered immediately, rather it's more of store-and-forward approach, similar to e-mail.

## Transport Layer: 
### Segmenting Protocols
+ TCP Transmission Control Protocol: connection oriented, which means it is reliable, if you send a segment it'll wait for an acknowledgment that the receiver indeed receive the segment.      
+ UDP User Datagram Protocol: connectionless, which means it is not reliable, it just keeps sending segments, doesn't use acknowledgment to make sure that the receiver indeed receive the segment. great for VOIP...      

!Note: the Maximum number of bytes that we can send in a single window is 2_000_000 Bytes,

## Common TCP/IP protocols:
Many of the application layer's protocols of the TCP/IP stack, have unique port number, so we can identify the by the port number.
To name few of these protocol and their port-number:
+ FTP (File Transfer Prtc):21: authenticated but not encrypted 
+ SSH (Secure Shell):TCP/UDP/SCTP port 22: authenticated and encrypted FTP (mix of FTP & SSH)
+ SFTP (SSH File Transfer Protocol Although many think "Secure File Transfer Protocol"):22: authenticated and encrypted
+ SCP (Secure Copy Prtc):22: authenticated and encrypted, 
+ TELNET (Telephone Net):23, can be authenticated not encrypted 
+ SMTP (Simple Message Transport Prtc):25 
+ DNS (Domain Name Server): 53
+ DHCP (Dynamic Host Configuration Prtcl): UDP:67
+ HTTP (Hyper Text Transfer Prtcl):80

> [!Note] when a client connects to a server it uses the protocol port number as its Destination Port, and a high number port number (Ephemeral port) as its Source Port,


> [!Note] Ephemeral ports: client's source ports number greater than 1023



