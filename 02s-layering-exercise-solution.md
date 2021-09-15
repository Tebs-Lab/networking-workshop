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

> You may want to convert between hex, decimal, and binary throughout this exercise. This website provides such a utility: https://www.rapidtables.com/convert/number/hex-to-decimal.html (there are many similar tools.) You may also want to convert between hex and ASCII (although xxd and format-hex do this conversion by default). This website does hex to ASCII: https://www.rapidtables.com/convert/number/hex-to-ascii.html

### Solution to 1:

The data should look like this

```
0000   3c 28 6d bb 10 55 8c 85 90 bc fe 72 08 00 45 00   <(m..U.....r..E.
0010   00 82 00 00 40 00 40 06 6b 13 c0 a8 56 1d ac d9   ....@.@.k...V...
0020   0b c4 ea 75 00 50 18 56 9b 62 9b 91 76 0d 80 18   ...u.P.V.b..v...
0030   08 0c e1 b5 00 00 01 01 08 0a 1d cb 12 dc 96 69   ...............i
0040   60 59 47 45 54 20 2f 20 48 54 54 50 2f 31 2e 31   `YGET / HTTP/1.1
0050   0d 0a 48 6f 73 74 3a 20 77 77 77 2e 67 6f 6f 67   ..Host: www.goog
0060   6c 65 2e 63 6f 6d 0d 0a 55 73 65 72 2d 41 67 65   le.com..User-Age
0070   6e 74 3a 20 63 75 72 6c 2f 37 2e 36 34 2e 31 0d   nt: curl/7.64.1.
0080   0a 41 63 63 65 70 74 3a 20 2a 2f 2a 0d 0a 0d 0a   .Accept: */*....
```

Critically, broken up into its separate sections...

The Link Layer (Ethernet II):

```
3c 28 6d bb 10 55 8c 85 90 bc fe 72 08 00
```

The Network Layer (IPv4):

```
45 00 00 82 00 00 40 00 40 06 6b 13 c0 a8 56 1d ac d9 0b c4
```

The Transport Layer (TCP):

```
ea 75 00 50 18 56 9b 62 9b 91 76 0d 80 18 08 0c e1 b5 00 00 01 01 08 0a 1d cb 12 dc 96 69 60 59
```

The Application Layer (HTTP 1.1):

```
47 45 54 20 2f 20 48 54 54 50 2f 31 2e 31 0d 0a 48 6f 73 74 3a 20 77 77 77 2e 67 6f 6f 67 6c 65 2e 63 6f 6d 0d 0a 55 73 65 72 2d 41 67 65 6e 74 3a 20 63 75 72 6c 2f 37 2e 36 34 2e 31 0d 0a 41 63 63 65 70 74 3a 20 2a 2f 2a 0d 0a 0d 0a
```


### 2. Annotate and Parse The Data

For this section, you may have to perform some research to find out certain details about protocol header lengths and values.

#### Data Link Layer

The first 14 bytes of this file represent an Ether II header. 

```
Dest: 3c 28 6d bb 10 55
Src: 8c 85 90 bc fe 72
Ether Type: 08 00
```

Can you:

* Identify the source and destination MAC address?
    * The first 5 bytes are the destination, the next 5 bytes are the source.
    * Destination: 3c 28 6d bb 10 55
    * Source: 8c 85 90 bc fe 72
    
* Identify the network layer protocol and which bytes within these first 14 indicate that protocol?
    * The 2 bytes following the source MAC are: 0800
    * Look that up here: https://www.iana.org/assignments/ieee-802-numbers/ieee-802-numbers.xhtml
    * 0800 --> IPv4

#### Network Layer

Now that you've identified the network layer protocol, can you...

```
45 00 00 82 00 00 40 00 40 06 6b 13 c0 a8 56 1d ac d9 0b c4 

==>

45 00 00 82 
00 00 40 00 
40 06 6b 13 
c0 a8 56 1d 
ac d9 0b c4

==>

Version: 4
IHL: 5
TOS: 00
Total Length: 00 82
ID: 00 00
Flags: top three bits of 4. 4 == 0100, so flags=010
Fragmentation Offest: 40 minus the top three bits, 40 => 0100 0000 => 00000 (aka no fragmentation)
TTL: 0x40 => 0d64
Protocol: 06 (TCP)
header checksum: 6b 13
Sourcee Address: c0 a8 56 1d => 192.168.86.29
Destination Address: ac d9 0b c4 => 172.217.11.196
```

* Identify the "header length" of this header?
    * The first byte in this header (`0x45`) represents the version (4) and the header length (5).
    * The 5 specifically means "The number of 32-bit words in this header."
    * 32 bits is 4 bytes, `5*4 = 20` bytes (or `5*32 = 160` bits).

* Identify the destination address?
    * The destination IP address is the final 4 bytes of the IP header, specifically: `ac d9 0b c4`
    * Interpreting these each as decimal numbers you get something more familiar: `172.217.11.196`

* Identify the size of the data according to this network layer header?
    * This is encoded in the 3rd and 4th bytes of the IP header: `00 82`
    * As a decimal number: 130 (this is in bytes)
    * Passes a gut check: total size of the raw packet is 144 bytes, and the first 14 are the data-link layer info.

* Identify the "Time to Live" value?
    * 5 bytes after the total size value, as hex `40`.
    * This is 64 in decimal.

* Identify the transport layer protocol?
    * Immediately following the Time To Live field, `06` (which is also 6 in decimal)
    * Look it up here: https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml
    * 6 is TCP!

#### Transport Layer

Now that you've identified the transport layer protocol, can you...

```
ea 75 00 50 18 56 9b 62 9b 91 76 0d 80 18 08 0c e1 b5 00 00 01 01 08 0a 1d cb 12 dc 96 69 60 59

==>

ea 75 00 50 
18 56 9b 62 
9b 91 76 0d 
80 18 08 0c 
e1 b5 00 00 
01 01 08 0a 
1d cb 12 dc 
96 69 60 59

==>

src port: ea 75 (60021)
dst port: 00 50 (80)
sequence num: 18 56 9b 62 (408329058)
ack num: 9b 91 76 0d (2610001421)
data offset: 8 (8)
reserved bits: top three bits of 0 => 0000 => 000
control flags: bottom 9 bits of 0x018 => 0000 0001 1000 => 0 0001 1000 (PSH, ACK)
Window Size: 08 0c (2060)
Checksum: e1 b5
Urgent Pointer: 00 00

Options:
01 01 08 0a 
1d cb 12 dc 
96 69 60 59

The first two (01 01) are both "no op" for padding the total length to be a factor of 4 bytes.

The remaining 10 bytes are a TCP Timestamp option:

.. .. 08 0a 
1d cb 12 dc 
96 69 60 59

This is used (in part) to measure round trip time. Each host picks a random number,
then adds the time elapsed since the last packet and adds it to that number each time
a new packet arrives/is sent. Alone in one packet though, they are meaningless. If we 
saw the NEXT packet made by this host we could derive some info about the RTT.

Kind (of option): 08 (timestamp)
Length: 0a (10)
TS value: 1d cb 12 dc (499847900)
TS echo: 96 69 60 59 (2523488345)
```

* Identify the source and destination ports?
    * Source: `0xea75` => 60021
    * Destination: `0x0050` => 80

* Given the destination port, can you guess the application protocol?
    * 80 is most commonly associated with HTTP

* Identify the length of this header (according to the header)?
    * This is the first half of a byte, following the sequence numbers and ack numbers. 
    * `0x8` => 8 32-bit words => 8*4 bytes => 32 bytes

* This protocol does not provide specific information about which protocol is used for the payload (aka, which application layer protocol is used). How do you suppose the application that receives this payload makes this determination? 
    * The program running at that port is expecting a particular kind of data (or perhaps a small handful of types). It is the applications responsibility to parse the remaining data and it essentially expects only some kinds of data to be sent. It parses the data according to those rules, and if something goes wrong the program errors.
    * In our case, with HTTP/1.1 data, the first space-separated chunk of text indicates the "method" and the second represents the protocol and version. A web server would try to parse this data as ASCII text and extract these values. If they were wrong, the server would respond with an error (probably a 422 or 415).
    * https://www.restapitutorial.com/httpstatuscodes.html

#### Application Layer

```
47 45 54 20 2f 20 48 54 54 50 2f 31 2e 31 0d 0a 48 6f 73 74 3a 20 77 77 77 2e 67 6f 6f 67 6c 65 2e 63 6f 6d 0d 0a 55 73 65 72 2d 41 67 65 6e 74 3a 20 63 75 72 6c 2f 37 2e 36 34 2e 31 0d 0a 41 63 63 65 70 74 3a 20 2a 2f 2a 0d 0a 0d 0a

==>

GET / HTTP/1.1
Host: www.google.com
User-Agent: curl/7.64.1
Accept: */*
```

* Using any method available, translate these bytes to ASCII.
* This application layer protocol identifies itself. What protocol is it, and what version?
    * HTTP 1.1

* Having done that, can you identify what URL was used when making this request?
    * www.google.com

* Can you identify what program made the request?
    * It was the program `curl`