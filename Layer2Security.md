<!-- Layer2Security.md -->

>Note: the switchports by defualts are 'dynamic auto'

# MAC flooding (aka CAM table overflow attack): 
MAC flooding is an attack where the attacker floods the switch with MAC addresses, knwoing that all switches have a limited number of MAC addresses that they can lean off of all their ports, and when that number is reached the switch will turn each VLAN into a HUB, broadcasting all frame of one VLAN over all ports belonging to that VLAN.
> show mac address-table count !display the maximum number of MAC address the switch can learn.

# Port security: 
Port security is the solution for theses Layer Two attacks.
A typical user uses just a single MAC address. Exceptions to this may be a device running virtual machine or two that might use different MAC addresses than their host, or if there is an IP phone with a built-in switch, which may also account for additional MAC addresses.

>Note: By default, port security will make sure that ONLY ONE MAC address will be allowed access on each switch port. You can set the maximum number of addresses in the range of 1 to 1024.

Each interface using port security dynamically learns MAC addresses by default and expects those addresses to appear on that interface in the future. MAC addresses are learned as hosts transmit frames on an interface. The interface learns up to the maximum number of addresses allowed. LEARNED ADDRESSES ALSO CAN BE AGED OUT OF THE TABLE IF THOSE HOSTS ARE SILENT FOR A PERIOD OF TIME. BY DEFAULT, NO AGING OCCURS.

The MAC address is given in dotted-triplet format. If the number of static addresses con- figured is less than the maximum number of addresses secured on a port, the remaining addresses are learned dynamically. Be sure to set the maximum number appropriately.

# Violation modes:
+ Shutdown: ( is the default violation mode) The port immediately is put into the errdisable state, which effectively shuts it down. It must be reenabled manually (must shut, and no shut) or through errdisable recovery to be used again.

+ Restrict: The port is allowed to stay up, but all packets from violating MAC address- es are dropped. The switch keeps a running count of the number of violating packets and can send an SNMP trap and a syslog message as an alert of the violation.

+ Protect: The port is allowed to stay up, as in the restrict mode. Although packets from violating addresses are dropped, no record of the violation is kept.

+ Shutdown VLAN "vlan":This mode mimics the behavior of the shutdown mode but limits the error disabled state the specific violating VLAN.

>Note: When you configure an interface for port-security and the violation as restrict and when the maximumn number of MAC exceeded violation occurred, you will get the following log:
%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address 0000.5e00.0101 on port GigabitEthernet1/0/11.

# To display the port security configuration on an interface
`#show port-security int "inter"`
port-security: Enabled !if the port-security is enabled
port-security: secure-up !if the port-security is enabled and the port is up, else secure-shutdown

>Note: shuting, and un-shutin the port after a violation occurred cleans up the violation counter.



# Set the port-security aging time
community.cisco.com/t5/switching/port-security-aging-time-what-is-it-good-for/td-p/1864366
switchport port-security aging ? {static, time, type}
switchport port-security aging type? {absolute, inactive}

# There are two types of aging you can configure:
1.)absolute—the secure addresses on that port are deleted after the specified aging time.
2.)inactivity—the secure addresess on this port are  deleted only if the secure addresses are inactive for the specified  aging time.
you have "inactivitiy" configured, which means that after 2 min(the time you have specified) the secure mac addresses are deleted.
this feature is useful if you want to grant access only for a certain time.


# Mac-address forbidden
Sets the MAC address forbidden on the interface.
Switch(config-if)# switchport port-security mac-address forbidden
Or, Sets the MAC address forbidden on all interfaces, globally.
Switch(config)# port-security mac-address forbidden
To verify the MAC addresses forbidden on the interface, use the show port-security address forbidden command, in privileged EXEC mode.
