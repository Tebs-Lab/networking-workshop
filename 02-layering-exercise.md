# Unpacking a Packet

In this exercise you'll be asked to look at the raw binary data of a single packet, and asked to identify certain features of the packet. The raw packet data is in the file 02x-raw-single-packet.

## Your tasks:

### 1. Examine the data.

On Mac or Linux, use the `xxd` command:

```
xxd 02x-single-raw-packet.bin
```

On Windows:

```
format-hex 02x-single-raw-packet.bin
```

The resulting information will be in a format called "hexedecimal" where every byte is separated from its neighboring bytes by a space, and every character represents 4 bits of data. 

> You may want to convert between hex, decimal, and binary throughout this exercise. This website provides such a utility: https://www.rapidtables.com/convert/number/hex-to-decimal.html (there are many similar tools.) You may also want to convert between hex and ASCII (although xxd and format-hex do this conversion by default). This website does hex to ASCII: https://www.rapidtables.com/convert/number/hex-to-ascii.html


### 2. Annotate and Parse The Data

For this section, you may have to perform some research to find out certain details about protocol header lengths and values.

#### Data Link Layer

The first 14 bytes of this file represent an Ether II header. Can you:

* Identify the source and destination MAC address?
* Identify the network layer protocol and which bytes within these first 14 indicate that protocol?
* NOTE: The CRC checksum bytes are not included in this captured packet. This is because the network interface discards them at the OS level (and if they are invalid typically discards the entire packet).

#### Network Layer

Now that you've identified the network layer protocol, can you...

* Identify the "header length" of this header?
* Identify the destination address?
* Identify the size of the data according to this network layer header?
* Identify the "Time to Live" value?
* Identify the transport layer protocol?

#### Transport Layer

Now that you've identified the transport layer protocol, can you...

* Identify the source and destination ports?
    * Given the destination port, can you guess the application protocol?
* Identify the length of this header (according to the header)?
* This protocol does not provide specific information about which protocol is used for the payload (aka, which application layer protocol is used). How do you suppose the application that receives this payload makes this determination? 
    * Why did we write "application" rather than "computer"?

#### Application Layer

* Using any method available, translate these bytes to ASCII.
* This application layer protocol identifies itself. What protocol is it, and what version?
* Having done that, can you identify what URL was used when making this request?
* Can you identify what program made the request?