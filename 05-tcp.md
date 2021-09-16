# TCP and The Transport Layer

This section describes 

## Transport Layer: "service" vs "responsibility"

* Recall that the transport layer is responsible for managing host-to-host level communication. 
* Any kind of application layer data might be sent via any transport layer protocol
    * And similarly, any networking layer might be used...
    * The transport layer is agnostic to both the layers above and below it.

* **However**, not all application layer data is appropriate to send over any given transport layer protocol.
* This is because the transport layer doesn't really HAVE to do much at all.
    * In fact, the only "responsibility" of the transport layer is to provide each host with the ability to multiplex this data to the proper application running on that host.

* Some transport layer protocols provide a lot of services, but some only provide the absolute basics...

## UDP: The Minimal Transport Layer Protocol

* The User Datagram Protocol (UDP) is a minimal transport layer protocol, and you can tell it's minimal by looking at its header:

![](https://notes.shichao.io/tcpv1/figure_10-2_600.png)

* The ports provide the application multiplexing (each application running on a host is assigned a port).
* It also provides a checksum field, which you might think provides some data integrity, but it is often ignored in practice and filled with all 0's (intended to indicate that the checksum is not being used).
* That's literally it. UDP provides no guarantees. You send the datagram out into the net, and you just hope it works.
    * Despite this lack of insurance, it IS actually still popular in some cases.
    * Many of the assurances provided by a transport layer protocol like TCP can be built into the above layer protocols instead, so UDP is often used as a basic transport protocol for more advanced application layer protocols.
    * Also, in some cases, it's better to just drop a packet here and there rather than shoot for true reliability (streaming content and other realtime applications are sometimes managed this way).
    * plus, the underlying infrastructure has gotten quite good, so reliability at the transport can sort of 0be seen as "overkill" (but only sort of)

## TCP: More Robust Transport Layer Protocol

* In contrast to UDP, TCP provides a lot of assurances:

0. Via ports, TCP provides application multiplexing.

1. TCP is "reliable" which means we know that exactly the data we sent arrives at the other host.
    * We use sequence numbers, acknowledgments, and a per-packet checksum to provide this guarantee.
    * When we send a packet, we give it a number, when we receive a packet we send back an "acknowledgment"
        * If the checksum doesn't match, we don't send acknowledgement.
        * If we don't receive acknowledgments for a particular packet for a period of time, we resend it.
        * If we receive whats called a "duplicate acknowledgment" we also resend any missing data. 
        * Acknowledgment are typically "batched," meaning TCP does not wait for an ack on every transmission, and the reciever may wait for many segments to arrive then acknowledge them all at once. 

2. TCP provides some "flow control" mechanisms.
    * These are designed to allow the hosts to send data to each other at an appropriate speed so that neither host gets overwhelmed.
    * The values for the host-to-host connection speed are established during the "handshake"
    * These values basically indicate how much unacknowledged data can be transmitted by one host before pausing to wait for an ACK.

3. TCP provides "congestion control" mechanisms.
    * TCP is a "polite" protocol...
    * In addition to trying not to overwhelm the host, it tries not to overwhelm the nodes on the network as well.
    * If TCP detects that "packet loss" has occurred (via timeout or via duplicate ack) it slows down.
    * On the other hand, TCP constantly speeds up it's send rate if no packet loss is occurring in order to maximally utilize bandwidth.

4. Finally, TCP is "connection oriented" 
    * which means that the two hosts establish a connection.
    * Some values associated with the connection are established, and each host maintains some state information related to the connection.
    * For example, sequence numbers, window sizes, and port numbers are all part of this connection data.

Lets look at the TCP header:

![](https://www.lifewire.com/thmb/uOXb1Mr4IVWF2H8FBYl5PbC1JAo=/774x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/tcp-headers-f2c0881ea4c94e919794b7c0677ab90a.jpg)

* Ports are used for application multiplexing, and many port numbers are "revered" for use by specific protocols.
    * 80 for HTTP, 223 for HTTPS
    * 22 for ssh, scp, sftp
    * 53 for DNS, 853 for DNS over TLS
    * and many more... https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers

* Sequence and Acknowledgment numbers are initialized randomly by each host 
    * After that, the increase by the size of the data sent in each segment.
    * Therefore, the seq and ack numbers imply both order and size of the underlying data.
 
* Data Offset indicates where the payload data starts (and therefore indicates the size of the "options" section)

* Reserved bits: used to ensure 32-bit alignment, these should always be 0.

* Control Flags: There are 9 flags that can be on or off on a per segment basis. 
    * Info about each one: https://www.keycdn.com/support/tcp-flags

* Window Size: used as part of the control flow mechanism. 
    * Indicates how much unacknowledged data can be "on the wire" before pausing the connection.
    * This can and is updated by TCP throughout the lifecycle of an individual connection.

* Checksum: a computed value designed to ensure data reliability. 

* Urgent pointer: usually 0, often ignored, but is intended to give TCP the ability to prioritize a subset of the data.

* TCP optional data: There are a lot of additional options that aren't required. 
    * See here: https://www.iana.org/assignments/tcp-parameters/tcp-parameters.xhtml