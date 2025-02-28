#Networking 
# IPv6

[I spent a WEEK without IPv4 to understand IPv6 transition mechanisms](https://youtu.be/e-oLBOL0rDE)


IPv6 : is 128 bit address that meant to solve the problem of the expiration of the ipv4 in the future, represented as 32 hex digit, and usually abbreviated (compressed)
+ No-broadcast: All node multicast if needed to broadcast.
+ IPv6 has simplified header, 5 fields instead of 12 fields, 
+ IPv6 also desinged with Security and mobility features built-in the protocol.
+ IPv6 abandon the concept of fragmentation, MTU discovery is performed for each session.
+ IPv6 can coexist with IPv4 during migration: Dual stack
+ IPv6 traffic can go over IPv4 by creating a tunnel

# IPv6 can be abbreviated, and there are rules for doing so, 
- leading zeros in each field can be omitted 
- contiguois field containing all zero can be represented with double colon. (only once of an IP) 

eg: 
2345:0123:4040:0000:0000:0000:000A:000B -> 2345:123:4040::A:B
2000:0000:0000:0000:1234:0000:0000:000B -> 2000::1234:0:0:B

# like IPv4 there are different types of IPv6

|       IPv6               |    IPv4       |
|--------------------------|---------------|
| Global Unicast           |  Public IP    |
| Multicast                |  Multicast    |
| Link local               |  APIPA        |
| Unique local             |  Private IP   |
| Loopback                 |  Loopback     |
| Unspecified              |      N/A      |
| Solicited-node Multicast |      N/A      |



- Global Unicast           : Public IP 
- Multicast                : Multicast  
- Link local               : APIPA      
- Unique local             : Private IP 
- Loopback                 : Loopback   
- Unspecified              : Unspecified IP of the sender
- Solicited-node Multicast : Multicast          

# Global Unicast: is ONE TO ONE communication
Global : because it's routable in the public internet
```
-----------------------------------------------------------------
|001| Global Routing Prefix | Subnet ID |       Interface ID     |
-----------------------------------------------------------------
| 3 |         45            |    16     |           64           |  :  Bit
-----------------------------------------------------------------
```

# Multicast address : is ONE TO MANY communication.
```
 ----------------------------------------------------------------
|11111111| Flags| Scope|                 Group ID                |
 ----------------------------------------------------------------
|    8   |   4  |   4  |                   112                   |  :  Bit
 ----------------------------------------------------------------
```
- The first Byte: is all ones (FF in hex).   
- the Flags are called : 0RPT
- 0 : is reserved bit.
- R : if it's set to 1, P and T must also be set to 1, if those bits are set to one, this means that we are embedding the address of Randivous-Point.
- Randivous-Point : A router which forwards multicast traffic to routers asking to recieve the trafic, Randivous-Point is used with PIM.  
- PIM Protocol Independent Multicast : a multicast routing protocol.

# Link Local address : 
Always starts with FF80::/10 and it is the APIPA counterpart in IPv4.
- can only comunicate on one network segment.
```
 ----------------------------------------------------------------
|1111 1110 10|    54 Zero  |           Interface ID              |
 ----------------------------------------------------------------
|   10 bits  |    54 bits  |               64 Bits               |
 ----------------------------------------------------------------
```
- IPv6 can use that link local address to do things like automatic address configuration, or to discover neighbors

# Unique Local address : always starts with FC00::/7 and it is the Private address counterpart in IPv4.
- can be routed only in private network but not the public internet.
```
 ----------------------------------------------------------------
|1111 111| L | Global ID | Subnet ID |       Interface ID        |
 ----------------------------------------------------------------
| 7 bits | 1 |  40 bits  |  16 bits  |         64 bits           | : Bit
 ----------------------------------------------------------------
```
- if the L bit is set this means that the address is locally assigned, and this will always be the case. 

# Loopback address: written as ::1
```
 ----------------------------------------------------------------
|                        127 ZEROS                            | 1|
 ----------------------------------------------------------------
```

# Unspecified address: written as ::
```
 ----------------------------------------------------------------
|                        128 ZEROS                               |
 ----------------------------------------------------------------
```
