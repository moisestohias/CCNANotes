In Cisco switches, to find the MAC address that caused a port security violation, you can use the following commands:

### Step-by-Step Procedure:

1. **Check the Port Security Violation Information:**
   First, you need to check if a violation has occurred and get information about the violations. Use the following command:

   ```
   show port-security
   ```

   This will give you an overview of port security status, including the number of violations, secure MAC addresses, and the port on which violations occurred.

2. **Show Detailed Violation Information:**
   To view more detailed information on violations and to identify the specific MAC address that triggered the violation, use the command:

   ```
   show port-security address
   ```

   This will display the secure MAC addresses that are currently allowed on the ports, and it will show if there have been any violations for specific ports.

3. **Check the Violation Log:**
   If you have logging enabled, you can also check the system logs for any violation messages. Use the following command to see the logs:

   ```
   show logging
   ```

   Look for entries related to port security violations. The log message will typically mention which port and which MAC address triggered the violation.

4. **Monitor Real-Time Violations (if violations are ongoing):**
   If you want to monitor for violations in real-time, you can enable `debug` for port security violations:

   ```
   debug port-security
   ```

   This will show live updates about any port security violations, including the MAC address that caused the violation.

5. **Check the Violation Count per Port:**
   You can also check for the number of violations on a particular port by using:

   ```
   show interface [interface-id] switchport
   ```

   This will show detailed switchport information for the specific interface and, if a violation has occurred, it will show the violation count and port security settings.

### Example of output after a violation:

If a violation occurs, the logs might look something like this:

```
%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address 00D0.1234.5678 on port Fa0/1
```

This log entry tells you that the MAC address `00D0.1234.5678` caused the violation on port `Fa0/1`.

### Conclusion:
By using the `show port-security address` and `show logging` commands, you can identify the MAC address responsible for a port security violation on a Cisco switch. Additionally, monitoring the logs or enabling `debug` will help you pinpoint violations in real-time.