Serial Ports.txt
#IT 

## More resoureces:
RS 232 DB9 Connector by Prof.Sachin Sambhaji Patil
youtube.com/watch?v=KeGteOm9I4Y
Explaining The Basics Of RS-232 Serial Communications
youtube.com/watch?v=XVEnxipCIJ0
What is RS232 and What is it Used for?
youtube.com/watch?v=eo9dbnrpspM


A serial interface is an interface that transfers data one bit at a time. In the early days of computing, pretty much every computer came with one or two serial ports - generally, one small and one large one.

There are many devices in a computer that use serial communication. A good example of this is the Universal Serial Bus or USB. There is, however, only one type of connection that is referred to as a serial port. Serial ports, built based on the RS-232 standard. This standard defines how data is sent over the port. For this reason, you may hear a serial connection referred to as an RS-232 connection. They are also commonly referred to as communication ports or COM ports for short.

>Note: In RS-232, user data is sent as a time-series of bits. Both synchronous and asynchronous transmissions are supported by the standard.

The serial ports themselves are called DE-9 and DB-25. The D refers to the shape of the connector. The letters E (Smaller) and B (Larger) refers to the size of the connector. The number at the end refers to the number of pins in the connection.

To fully meet the RS-232 standard, the serial port required 25 pins. Both connectors DE-9 and DB-25 use serial communication. Since serial communication is used, this essentially means that only two pins are required for data transfer - one to transmit data and one to receive data. Thus, both connectors have the same number of pins used for data. What is different, however, is the larger connector has more pins that are used for signaling.

Even back when serial ports were being used, there were very few devices on the market that required the extra pins for signaling. For this reason, generally as time passed, only the smaller serial ports were used. To implement the larger serial port cost more money and, if the functionality was not being used, it was a waste of time.

As time passed, some computers would have two smaller serial ports. Later still, the serial ports on computers started to decline, so only one of the smaller serial ports was included. Nowadays, you won’t find a serial port on a computer. Some motherboards will have a header where a serial port can be added, otherwise they won’t have anything. As time goes on, it is expected that the serial port will disappear from the computer all together.

Nowadays, generally the only time you will use a serial port is when working with old equipment or accessing certain hardware devices. Let’s have a look at how you may go about doing this.

Serial Port Access
2:50 If your motherboard has a serial port header, you can purchase a bracket with a cable that will plug into the header on the motherboard. It is just a matter of installing the bracket in a free expansion slot on the motherboard.

You can also purchase a USB to serial port cable. This is the easy option nowadays. Given that the administrator will rarely need access to a serial port, the USB cable is something the administrator can keep around for when they need it. Often a technician will use a serial port to access equipment. This will often be done just out of convenience using a laptop. A bracket cannot be installed in a laptop so the USB to serial cable is a good option.

In some cases, you may need to change the serial port from one type to another. To do this, you can use an adapter. This adapter will convert DE-9 to DB-25. You can also get adapters that change the gender of the connection. Sometimes this may be useful. Nowadays, generally the USB to serial cable will work in most cases.

Now let’s have a look at how you would access a device using the serial port.

Demonstration
3:58 There are many different software programs that can be used to access the serial port. In this demonstration I will use some free software called Putty. Putty can always be used for secure shell, telnet and serial connections. It has a lot of features and is free, so it is good software for the administrator to have.

Serial connections are not automatically configured like a lot of modern computer components. In most cases, a serial connection is a last resort. If you have another way of accessing the device, I would use that.