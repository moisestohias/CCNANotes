#Networking 
# Routing

# IP
## Binary:
+ Understanding Binary is critically important to understand how Subneting and IP address works.
+ Binary Conversion is the most valuable skill you need to acquire before learning about Subneting.
bit : is a combination of Binary digIT.
Binary: literally means out of two state.
to convert from decimal to binary and vice versa is easier to use Binary Conversion Table:


| 128 |  64 |  32 |  16 |  8  |  4  |  2  |  1  |
|-----|-----|-----|-----|-----|-----|-----|-----|
| 0   |  1  |  1  |  0  |  1  |  0  |  1  |  1  |

64 +  32 +  8  +  2  +  1  = 107


You must memoirize the following values, to be really effective in subneting.
+ 240 - 1111 0000
+ 224 - 1110 0000
+ 208 - 1101 0000
+ 192 - 1100 0000
+ 176 - 1011 0000
+ 160 - 1010 0000
+ 144 - 1001 0000
+ 128 - 1000 0000

## IPver4 Format
IPver4 is 4 Byte = 32 bit, grouped in 8 bit = 1 Byte, this format is called the Doted Decimal Notation, some of the bits represent the network, the remaining bits represent the device on the network, we do that using the Subnet mask which draw that dividing line.

## Subnet Mask
Subnet Mask is 32 bit (in IPver4) binary string that divides the network bits from the host bit.
> there are two notation commonly used for representing the Subnet mask:
    - 10.1.0.2/8 : slash notation aka prefix notation
    - 10.1.0.2 255.0.0.0 : dotted decimal notation pretty much like the IP.

> the Network address is calculated by setting all the host bits of the IP to 0,
    - 10.0.0.0
> the Host address is calculated by setting all the network bits of the IP to 0,
    - 0.1.0.2

- classfull Masks aka Default Mask,also Natural Mask.

# IP Address Classes:
+ **Class A,B and C**: are the only classes that's going to be assigned to host, and has classfull masks.
+ **Class D**: is for multicast.
+ **Class E**: is for experimental purposes, never used actually.

| class | range     |  default mask  |
|-------|-----------|----------------|
| A     |   1 - 126 |  255.0.0.0     |
| B     | 127 - 191 |  255.255.0.0   |
| C     | 192 - 223 |  255.255.255.0 |
| D     | 224 - 239 |                |
| E     | 240 - 254 |                |


> [!Note] changing the default mask doesn't change the class of the IP. The class of the IP is entirely determined by the value of the first octet, no matter how long that mask is.


+ 127.0.0.0 to 127.255.255.254 is LOCAL LOOPBACK IP address of a host, is not routable and used internally for testing the TCP/IP stack, so you can ping it to test your local TCP/IP stack is working correctly.

+ ICANN International Corporation for Assigning Names and Numbers, it's non profit organization for distributing range of IP addresses to international registry.
ARIN (Americal Registery For Internet Numbers): is the regional north american organization for assigning IP addresses.

# Private IP address

| address class |         address range         | default mask  |
| ------------- | :---------------------------: | :-----------: |
| **A**         |   10.0.0.0 - 10.255.255.255   |   255.0.0.0   |
| **B**         |  172.16.0.0 - 172.31.255.255  |  255.255.0.0  |
| **B**         | 169.254.0.0 - 169.254.255.255 |  255.255.0.0  |
| **C**         | 192.168.0.0 - 192.168.255.255 | 255.255.255.0 |


the range of : 169.254.0.0 - 169.254.255.255 is know as APIPA Automatic Private IP Address.

>Note: when a device don't get an IP address from the DHCP server it assign it self APIPA which is not routable, even in your local network, it only do so  so it can speak IP, namely A and other layer 2 protocols, it's generally not good thing to have an APIPA, and this host need attention.

## Trfic Types
+ **UNICAST** : one to one.
+ **MULTICAST** : one to many. devices can join a Multicast Group.
+ **BRAODCAST** : one to all.


## Subneting

| len | dec | bin       |
| --- | --- | --------- |
| /30 | 252 | 1111 1100 |
| /29 | 248 | 1111 1000 |
| /28 | 240 | 1111 0000 |
| /27 | 224 | 1110 0000 |
| /26 | 192 | 1100 0000 |
| /25 | 128 | 1000 0000 |


# Available Subnets
from design perspective we should be able to calculate the number of bits that we need to borrow to add to the natural mask, so we be able to create the number of subnet that we need, the formula for that :
Number of created Subnets = 2^s, where s is the number of borrowed bits.

## Available Hosts
+ Number of usable IP address = $2^h - 2$, where h is the number of Host bits.
+ The Interesting Octet : is the last octet that contain one in the subnet mask
+ The block size is the increment that we gonna count by in the interesting octet.
+ Number Block Size = 256 - value of the subnet mask's interesting octet
