# Module 2 Router Operation Introduction ######################
#STP #SpanningTreeProtocol #Cisco #Networking #IT

Privileged EXEC leve 15
User EXEC leve 1

= Lesson 1 Understanding a Router's Boot Sequence ===================
** Standard Boot**
-> POST (Power On Self Test): piece of code that lives in the ROM.
-> Bootstrap : piece of code that lives in the ROM, responsible for the next two steps, Locating IOS, Loading it to the RAM.
-> Locating the IOS.
-> Loading the IOS to the RAM.
-> Locating the startup-configuration.
-> Loading the startup-configuration to the RAM.
-> Executing the configuration (bringing interfaces up, IPs, RoutingP ..).

= Lesson 2 Differentiating Between Boot Options  ================
- There are three Boot options:
=> 1 (0x2100): ROM monitor: very basic OS that lives in the ROM, the router wont able to do most of it's functionality but it can bring a interface up and assign IP to it, so it can be used in case of the corruption of flash to download new IOS in the newly installed flash from TFTP server, or used in case of password recovery , or to change the baud rate.
=> 2 (0x2101): Boot from the first IOS image in the flash
=> 3 (0x2102 basicallly anvalue between 2-F): get image loading instruction form the start-config file stored in NVRAM
> show run-config : boot system represent the image loading instruction   

>NOTE: The configuration Register is what decides which boot option the router boots in, more specifically is the last hex digits (4bits) of the config-reg.
0x0 > ROM monitor mode
0x1 > first IOS image in the flash
0x2-0xf > get image loading instruction form the startup-configuration file in NVRAM, if no image loading instruction are found, the device will load the first IOS in the flash, if it doesn't fine one, it will attempt to download an image from TFTP server on the network, with the destination IP address of 255.255.255.255, if it can't find TFTP server or a bootable image, after 6 unsuccessful attempt it will try to boot in ROM monitor mode.   

=> The configuration Register : 16 bits value, written in hexadecimal that lives in the NVRAM (not in the startup-configuration file but on its own), and can be changed from the global configuration.
- The default value for most cases in most devices is : 0x2102, which tells the router to get its image loading instructions form the startup-config file in NVRAM.

> show version : very last line displays the configuration Register
>(config)# config-register 0x2100 : to go the ROM Monitor mode next boot

>Note: in the ROM Monitor mode, 'confreg' is the command to set the config-register, and after you done use 'reset' to reboot the device. 


read more about Configuration-Register on CISCO website -----------------
https://www.cisco.com/c/en/us/support/docs/routers/10000-series-routers/50421-config-register-use.html
https://www.cisco.com/E-Learning/bulk/public/tac/cim/cib/using_cisco_ios_software/05_config_register.htm
--------------------------------------------------------------
Bit No : Hex : Value Meaning/Function
00-03  0x0000 to 0x000F : Defines the source of a default Cisco IOS Software image required to run the router:

00    : Stays at the system bootstrap prompt.
01    : Boots the first system image in onboard Flash memory.
02-0F : Specifies a default netboot filename. Enables boot system commands that override the default netboot filename.
04 : not used
05 : along with 11 and 12 define cosole speed,
06    0x0040  Causes system software to ignore NVRAM contents.
07    0x0080  Enables the original equipment manufacturer (OEM) bit.
08    0x0100  Disables the Break function.
09    0x0200  Uses secondary bootstrap.
10    0x0400  Broadcasts Internet Protocol (IP) with all zeros.
11-12 0x0800  to 0x1000  Defines the console baud rate (the default setting is 9600 baud).
13    0x2000  Boots default Flash software if network boot fails.
14    0x4000  Causes IP broadcasts to leave out network numbers.
15    0x8000  Enables diagnostic messages and ignores the contents of NVRAM

*Note : you dont need to save the run-config as the startup-config, because once you press Enter, the new value will be saved as config-register value in the NVRAM, bc i "believe" the Conf-Reg is detached from config-file


= Lesson 3 Working with CISCO IOS Files =======================
- CISCO routers and switches have their own Integrated File System to store files (OS) in the flash memory of the device , and the IFS can use a TFTP server (which can be other devices) to download IOS to the flash, and also it's used to store and access file from and in the NVRAM, and also supports FTP which can uses as well to store a backup copy of the configuration.
> show file system : displays storage devices that the FS can expliot
> show flash : take you to the flash and displays the content of it
>Note: you can navigate the FS using LINUX aliases: cd, dir, mkdir, cp, mv, more...
e.g: 
- to save the output of one command to a file on the flush, like show ip int brief , you can pip it and redirect it to a file on the flash:
> show ip int brief | redirect flash:/outputfile.txt
> more flash:/outputfile.txt  !To display the content of the file.

- to copy the startup-configuration to a TFTP server you use the copy command, and you specify the destination as tftp , then you'll be prompted for the IP address of that server , then the output file name.
> copy startup-config TFTP "e.g:192.168.1.5" "file-name" 


= Lesson 4 CISCO IOS Images  ========================================
c2800nm-adventerprisek9-mz.151-4.M6.bin
c2800nm -> the Platform series.
adventerprisek9 -> (Advanced Enterprise) Feature Set  
mz -> M: the IOS Will be run in RAM, Z: compressed image  
151-4.M6 -> IOS Version (151 = 15.1)
bin -> file extension , bin == binary executable

-You can use the feature set navigator at CISCO website:
cfnng.cisco.com

==> Importing IOS via TFTP
To download an IOS image from a TFTP server we just copy the image from the TFTP to the the flash, using the copy command, which follows with a sery of instructions.   
> copy tftp: flash: 
> address or name of the remote host : "IP of server" 
> source file name : "IOS image file name" 

- the next step is to make the router boot from the newly downloaded image, we need to change the boot configuration in the startup-config by, removing the old IOS image from the config and adding the new IOS instead.
> R(config)# boot system flash:"NEW IOS file name" 
> R(config)# no boot system flash:"OLD IOS file name" 
> write then reload 

= Lesson 5 Cisco IOS Licenses  ====================================
- when you purchase a router device it comes with its license, if you want to purchase new license, CISCO will give you Software Claim Certificate which can be PDF that they email you, or a paper that contains a PKA (Product Authorization Key), then you can use this PKA to get a license file using:
- CISCO License Manager software that can be found in: cisco.com/go/clm
- CISCO License Registration Portal using this url: cisco.com/go/license

>Note: Licenses come in .xml file format that should be in the flash **

=> Licenses before Cisco IOS 15.0: there was five basic License
- IPBase : the basic feature set that all other license contain
- IPVoice : Voice-Over-IP, IP Telephony features
- Advanced security : Security features e.g : IPSec, IPS, IOS Firewall
- SP Services : Services provider features :MLPS, ATM
- Enterprise Base : support multiple protocols environment (IBM, AppleTalk, IPX..)
-> Premium Packages, which are basically a combination of the basic license:
- Advanced IP Services: Advanced Security & Service Provider feature set
- Enterprise Services: Enterprise Base & Service Provider feature set
- Advanced Enterprise Services: offers the Full set of CISCO features

=> Licenses Beggining with Cisco IOS 15.0: there is four basic Licenses
- IPBase (IPBaseK9): the basic set of CISCO IOS technologies
- Security (SeckK9): security feature IPsec, IOS firewall, IPS
- Unified Communication (UCK9): VoIP and IP telephone features (Voice port , transcoding)
- Data (DATAK9): service provider and multi-protocol support (eg; MPLS, IPX)

!Note: in all IOS version 15 and above, Images all feature sets are included, you only need the appropriate license to unlock them.  

> show license !displays the installed licenses and the available to evaluate

> install flash: "license name.xml" : to install a license from the flash
> license boot module "platform e.g c2900" technology-package "e.g UCk9, securityk9" : install evaluation license, the evaluation duration is 60 days.
> license save flash:"licence_backup_copy.xml" : saving license to the flash

=> Removing a license: before a license can be removed it should be disabled
> license boot module "e.g c2900" technology-package "e.g UCk9" disable
> reloade : the router should be reloaded before removing the license
> license clear "e.g UCK9"
> no license boot module "e.g c2900" technology-package "e.g UCk9" disable : to remove this 'license disable' entry from the running-config bc even after disabling it and clear it the command will stay in config-file as disable
> wr : save the running-config as the startup-config
> reloade : the router should be reloaded

= Lesson 6 Password Recovery ====================================
- to recover password in case you forget it, reboot the device and while it's booting you hit break command , and this will cause the router to enter the ROM Monitor mode, then from there you use confreg util to change the configuration-register to "0x2142", which will tell the router to ignore the startup-config
> confreg "0x2142"
> reset : to restart the router
- once you set the conf-reg to "0x2142" the router won't load the startup-config to the RAM, it enter as if it would have bieng the first time you boot the router, then you can enter global-config mode and copy the startup-config to the running-config (to RAM), "booom !!!", 
- then you set new password and reset the config-register back to "0x2102", btw doing this won't bring the interfaces up you have to do it manually 
- finally save the running-config as the startup-config

= Lesson 7 Routing protocols characteristics ====================================
show ip ospf rib !Not available in older IOSs namely 12.5 and bellow.
! displays the routing information database of OSPF, (routes OSPF knows about) 
show ip eigrp topology !Not available in older IOSs namely 12.5 and bellow.
! displays the routing information database of EIGRP, (routes EIGRP knows about) 
show ip cef
! displays the CEF table, the router currently using.
show adjacency
! displays adjacent devices IP, and the eagers interface to them
show ip arp
! displays the Arp cash table
ip routing
! enables IP routing, which is enabled by default on all routers, but not layer 3 switches
show ip protocols
! displays routing protocols used by this router


==> BFD
Bi-directional Forwarding Detection (BFD) is a Layer 2 detection protocol designed to provide fast forwarding path failure detection for Layer 2 redundancy protocols using Ethernet across all media types


Request access to CME 4.1
https://www.youtube.com/watch?v=iVOg-Cxn3bk

Windows tool to add network adapter: hdwwiz.exe, select network adapter, then Microsoft, KM loopback adapter

Misc Cisoc UCM
https://www.youtube.com/user/wmx99/videos