# 2 - Switch Configuration
## Lesson 1 Switch Hardware

We can get a quit a bit of information if we have a physical access to that switch in a form of LEDs, different models have different LEDs.

>Warning: You need to refer to the catalog of your device for specific information. But generally many of  CISCO switches use almost the same signaling.

## Catalyst 2960 series Switch LED:
### syst (system):
- **Off**   > the system is off
- **Amber** > malfunction occurred during the POST,
- **Green** >  the Switch is up,
### RPS (Redundant Power Supply):
- **Off**   > Either RPS is not installed, or it's not powered on.
- **Amber** > RPS is installed, but it's not operational
- **Blinking Amber** > primary power supply has failed, RPS taking place.
- **Green** > RPS is installed and it's operational, and ready to take over.
- **Blinking Green** > RPS is installed and it's operational. However it's powering another device, and therefor unable to power the switch.

### Mode : 
Mode toggle between different modes, so port's LED reflect different info
- **Status**
    - **Off** > Either no link present or administratively disabled.
    - **Solid Green** > link is up & good, aka LINK INTEGRITY.
    - **Blinking Green** > link is up & traffic is present on that port.
    - **Amber** > link is either administratively disabled or disabled due to layer 2 topological loop (STP) , or other layer 2 configurations.
    - **ALternating Amber & Green** > errors, e.g excessive collisions, CRC
- **Duplex**
    - **Off** : port is operating in HALF duplex mode
    - **Green** : port is operating in FULL duplex mode

## Lesson 66 Port Addressing
### Port Addressing:
The port addressing changes depends on the device you are working with, and modules installed on that device, but generally they all follow consistent naming scheme,
+ StackWise: allows stack of as many as 9 switches to be stacked over each other in the rack and managed as one unit.
For switch that support stacking, like the 3750 series switch. The port address have three number; Switch in Stack/module/PortNumber
+ Normal stand alone switch: Module/PortNumber
For the port that are built-in to the switch, which means are managed by the mother board, The Module is always 0 in this case. Because the Motherboard is considered 0.

# Lesson 2 Logging In
## Connecting via the Console:
If we want to connect to the switch for the first time, we need to have:
+ A physical access to the switch hardware,
+ A console cable aka 'Role over cable' (RJ-45 from one side that connects to the switch, and 9-pin serial connector (DE9 but commonly referred to as DB9) on the other side)

> [!Note] the name of the Roll-Over came from the fact if you want to make your own console cable you have to roll-over all the individual wires of the cable, unlike the corss-over cable Ethernet cables, the roll-over literally flips the all wire, 1->8, 2->7 ...
[More info about serial ports and connections](https://youtube.com/watch?v=O1MhwPb2NWA)

+ A Serial-to-USB adapter if the Lab-top doesn't have a serial port which is the case with all modern laptops, or if the switch has Mini-USB port we can use a USB-miniUSB cable and the appropriate driver to connect to the switch.
+ A terminal emulator e.g Putty, TeraTerm, SecureCRT.
Pinouts: is how the individual wires in the roll-over cable are are used.

> [!Note] This is critical, if you're planning on performing password recovery, you need a serial cable (roll-over cable) that uses the prolific chipset (costly compared to non-prolific one), which support the sending of a break signal.

> [!Note] The auxiliary port is used to connect to device remotely using modem, the aux cable must be connected to modem, you have to dial up the connection and this will give access remotely.

> [!Warning] the big difference between console and the aux port, is that the console port will be active during the router boot process, which is the key idea that allows for password recover, while the aux port will be active only after the device is booted. Another key difference is that the console port has no login enabled on it and privilege level is 15, the aux port on the other hand, has no password but its privilege level is 1. You will only be able to fully access the device for configuration and management after you configure a password because the "login"

> [!Note] if you are using USB to Serial adapter, in Windows go to device manager and look under serial connection, you will see a the COM port and its number, you need to use that com interface with that number if your terminal emulator.

##  Default Settings of a console port on Cisco devices:
|params       | default|
|-------------|--------|
|Bits per sec |  9600  |
|Data bits    |     8  |
|Parity       |  none  |
|Stop bits    |     1  |
|Flow control |  none  |


# Configuring a Management IP Address
In order to access the switch remotely, it has to have a Management IP, and default Gateway (just like a PC on the network), to give the switch Man IP we need to have a physical access so we can console to the switch and do that, because all the switchs are layer two device, and all of ports are supposed to connect to either hosts, or other network device, we need SVI (Switch Virtual Interface) to assign the mangement IP to it

`show line`: displays the connection lines available to connect to the device, the starred ones indicate the active line(s).

> [!Info] The switch by default has all of its port on one default VLAN vlan1, since that VLAN is built in and native it's perfect (for now) for it to have the management IP, so we need to assign the Management IP to that VLAN.

## to assigns the default VLAN the Man IP
```
(config)# interface "VLAN 1"
(config-if)# IP address "IP" "SubnetMask".
(config-if)# not shutdown: as a good practice because it might not the default VLAN and it's shutdown.
```

# Configuring a Default Gateway
To be able to access that device through a network this devise need default Gateway IP.

> [!Note] To assigns the default Gateway: IP default-gateway "IP"

+ `ip name-server server-ip-1 [server-ip-2 …]`: Global command. Configures the IPv4 addresses of DNS servers, so any commands when logged in to the switch will use the DNS for name resolution.

# Setting Console and VTY Passwords
> [!Note] the reason you can't telnet to a CISCO device until you assign a password to the VTY line, is because the login command is there by default (under the vty line), which require a login procedure to be taken, and there is no password present so the telnet will be prohibited. If you want to disable the login security prompt, you need to enter under the vty line "no login".

+ `show line` : display the connection lines that we can connect through to the device.
+ `password`  : assigns a password to a line, we need to select the line first : line console or vty line number.

> [!Note] when assigning a password to console line, make sure to enter the "login" command

We can assigns the same password to all line all at once: line vty 0 15

`(config)# line ?` : displays available commands. 
### console line :
+ `line console 0` : enter console configuration
    + `password "CISCO"` : set a password to the console line
    + `login` : Tells IOS to prompt for a password.
    + `login local`: Tells IOS to prompt for a username and password, to be checked against locally configured username global configuration commands

### vty lines
```
(config)# line vty 0 15 : configure vty lines all at once
(config-line)# password "CISCO" : set a password to the vty lines
(config-line)# login : enable password authentication
```

## Checking for Connectivity with Ping
Before you would be able to access the device remotely, you should first have connectivity to the device through the network, to check that you use the ping command from your PC to the Switch or Router,

### Enabling Telnet Access
> [!Note] before you would be able to telnet, the vty lines of the device should have a password and login in another word authentication enabled, and the privileged mode secured with a password either secret or password not sure exactly about the last one in all real devices but in packet tracer sure.

### Connecting to device via Telnet/SSH
to be able to TELNT or SSh to the switch, those input transport methods have to be allowed as an input in the vty line that we are connecting through,
## 
to see the allowed transport methods and to display other info of the line : show line vty #, down you'll find allowed inputs and outputs.
## we can manually configure the allowed inputs and outputs
> line vty 0 15
> transport input ? to see the available options, then we can select all: transport input all.

By default, you cannot telnet to a switch or router until you have configured a telnet password...

## Securing Access:
enable secret
service password-encryption: enable level 7 encryption on the password
## Configuring SSH
> [!Note] SSH connection requires the exchange of SSL certificate, and to generate SSL certificate, so to use SSH you need a domain name.
> [!Note] SSH also requires a user account (username and password), these user credential needs to be stored somewhere, for simplicity and most often for small businesses, these credential will be stored locally on the device itself. The 'username "user" password "pswd"' command will create a locally stored account.

+ First step change the hostname of the device to something other than the default, bc you will need to assign it a domain-name, and the domain-name has to be unique, and to ensure it is unique, CISOC devices will enforce you to change the name of the device.
+ Second assign the device a domain name: ip domain-name moises.io
crypto-key generate rsa modulus 1024

### Configuraing SSH
+ `username CISCO password CISCO` or # username CISCO secret CISCO
> [!Note]: username must be changed, to something other than the default, to in order to set local domain-name.

```
IP domain-name moises.com
crypto key generate rsa modulus 512 : generate the SSL certificate
IP ssh version 2
line vty 0 15
login local
```

> [!Note] the length of the key will ditermin the version of the SSH, 512 will result in SSH 1.5, 1024 will result SSH version 2

> [!Note]The domain-name is used for the construction of the digital certificate, when you go to website that is using HTTPS, the site sends you its digital certificate with the public key to encrypts your data with it, the only key that can decrypt the data, is the private key.

## login vs login local
+ `login` : Tells IOS to prompt for a password.
+ `login local`: Tells IOS to prompt for a username and password, to be checked against locally configured username global configuration commands

## configuring the history buffer size:
+ `history size <length>`: Line config mode. Defines the number of commands held in the history buffer, for later recall, for users of those lines.

## Disabling the consol logs:
+ `[no] logging console`:  Global command that disables or enables the display of log messages to the console.


## Show commands:
+ `show dhcp lease`: Lists any information the switch acquires as a DHCP client. This includes IP address, subnet mask, and default gateway information.
+ `show crypto key mypubkey rsa`: Lists the public and shared key created for use with SSH using the crypto key generate rsa global configuration command.
+ `show ip ssh`: Lists status information for the SSH server, including the SSH version.
+ `show ssh`: Lists status information for current SSH connections into and out of the local switch.
+ `show ip default-gateway`:
+ `show history`:
