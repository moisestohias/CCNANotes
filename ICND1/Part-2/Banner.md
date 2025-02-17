Cisco supports three types of Banners that can be used to display msg to the user accessing the device
-MOTD : Used for temporary messages that can change from time to time, such as “Router1 down for maintenance at midnight.”
-Login : is always shown before the user logs in, used to show warning messages, like “Unauthorized Access Prohibited.”
-Exec : always appears after login, it typically lists device information that outsiders should not see but that internal staff might want to know, for example, the exact location of the device.

# Banner display Oreder:
Based on the method used to access the device, the banners show up in slightly different order
Console and Telnet: MOTD,loging -> user-login -> Exec
SSH: Loging -> user-login -> MOTD, Exec
!Warning: when using SSHv1, the login banner is not shown to the SSH user.

# Banner Configuration:
The banner global configuration command can be used to configure all three types of these banners.
banner login "Unauthorized access is prohibited by law, those who ignore this clause and pass to further steps will be prosecuted by the power of law. "
