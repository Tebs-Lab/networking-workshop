# Layering and Separation of Concerns

The purpose of this section is to introduce the concept of network layering, as well as two explicit models of this layering: OSI and the simplified TCP/IP stack. 

## Layering: What and Why (High Level)

* In the world of networking a lot of work has been done to separate the various concerns of actually implementing a networked computer system. This is achieved through the concept of layering.

* Display the following image, or draw something similar.
    * These are two different conceptual models for achieving network layering.
    * Neither of these should be taken as a "gold standard," requirement, or anything like that. Instead, these are two ways to think about separating concerns.
    * In practice, the lines between the layers (especially in the top half of OSI) are extremely blurred, and increasingly so as computer networking matures as a field.

![](https://cdn.guru99.com/images/1/102219_1135_TCPIPvsOSIM1.png)

* One by one, describe the OSI layers from top to bottom.
* Physical layer: Data must be physically transmitted in some way, and MANY such mediums are supported:
    * Radio waves
    * Copper/Coax
    * Fiber optic
    * The physical layer represents this transmission mechanism, and only the mechanism.

* Data Link layer: These physical transmission mechanisms need supporting hardware and protocols.
    * Ethernet cables are a specific implementation of transmitting data through copper wires. 
    * WiFi is a specific implementation of utilizing radio waves.
    * The link layer *governs* the physical layer, as such it is a combination of hardware and software protocols.
        * e.g. Ethernet cables must be built to a specific standard, and an ethernet interface expects those cables to transmit data in a very specific way, governed by the ethernet protocol.
        * e.g. WiFi uses a particular spectrum of radio waves, and transmits them in very specific patterns. WiFi interfaces listen for these types of broadcasts and discard other types.

* Note that TCP/IP combines these two layers into one called the "network interface."
    * I think this is pretty fair, the link and physical media are pretty tightly linked in practice.

* The Network layer governs how data travels from point A to point B.
    * In cases where two computers are directly connected with an Ethernet cable, the network layer is not needed.
    * But, in modern networking use cases, we generally are accessing servers all over the world.
    * The networking layers responsibility is then to send data across several "data link" layer devices until it arrives at the destination.
    * By far the most common protocol used for this is the Internet Protocol (IP).
        * But some supporting algorithms are required to make this work, for example the Border Gateway Protocol.
    * Networking protocols assume that there are some data link and physical layer technologies that connect point A and point B, and can be agnostic of which technologies are actually used.

* The Transport layer is responsible for governing the transmission of data on a host-to-host level.
    * Meaning, Transport protocols can assume that a networking protocol has found a path from A to B, and the transport protocols can provide a supporting role.
    * Transport layer protocols CAN (but may not) provide services such as: 
        * Reliable transport (prove that the sender received what I sent).
        * Break larger transmissions into chunks.
        * Congestion control mechanisms (try not to overwhelm the network).
        * Flow control mechanisms (try not to overwhelm either of the hosts)
        * Route data to the proper process on each host.
    * TCP and UDP are among the most common protocols at this level, but this is changing in recent years with protocols such as QUIC gaining traction.
        * Same goes for bespoke protocols for certain workloads such as video streaming.
    * Transport layer protocols assume that the network layer works, that there IS some physical path for the data to travel, and that those links can all communicate properly.

* The Session Layer is responsible for managing connections ON AN APPLICATION LEVEL.
    * The transport layer may (or may not) establish connections at the HOST level.
    * The difference is between a session for two computers, which might be transmitting data related to multiple applications simultaneously, and a session between two specific applications on those two computers. 
        * The session layer governs how a SQL client and a SQL server communicate (specific to the SQL implementation), whereas the Transport layer governs how one computer communicates with another, but is agnostic about what specific data is being transmitted.

* The Presentation layer primarily governs file format information.
    * A picture can be a `gif`, `jpeg`, or `tiff` which could be considered three separate presentation layer protocols. 
    * Similarly audio could be a `wav`, `midi`, or `mp3`
    * On the web, `css` and `html` would also be examples of presentation layer protocols.

* Finally, the application layer governs how two applications communicate.
    * This layer has the most bespoke software, because individual applications may support completely custom features.
    * But, there are also some common standards such as HTTP. Essentially all web servers and web browsers utilize HTTP as a core part of their communication protocol.
    * DNS is another example that is well standardized and widely used.


* TCP/IP just bundles session, presentation, and application layer concerns into one layer.

* **Why perform this kind of layering?**
    * Separating concerns makes individual protocols easier to manage,
    * Separating concerns allows new technology to more easily replace old tech, by replacing individual layers on their own.
    * ... there could be lots of good answers here ...

## Layering in Practice

* An analogy: network layering is like sending a series of nested letters.
    * Each envelope contains instructions for a single person, and an additional envelope to be delivered to someone else.
    * Each person only needs to handle THEIR letter, and they don't even need to see the other letters, simply deliver them to the proper person.

* The reality by example...
* Lets say we're going to make an HTTP request from our web browser...

* **Micro Exercise: Demonstrate first, then have students do the same**
    * Everyone open up a web browser and open up the developer tools. 
    * One tab will be called the Network tab.
    * The first entry in the list will be the request to initially fetch the website, take a look at it... 
    * Make the same request in `curl`

Here's my CURL request header:

```
GET / HTTP/1.1
Host: www.google.com
User-Agent: curl/7.64.1
Accept: */*
```

and from Firefox:

```
GET / HTTP/2
Host: www.google.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:89.0) Gecko/20100101 Firefox/89.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
DNT: 1
Connection: keep-alive
Cookie: NID=223=hAlBYDix1ldZXcszdK6Fn9qbf5FxHz6uD1cFUWhGhPxwERlYEIeyCOe3E-jHFaEMIzcZ27xadzr2miFgkfmbOUDwYW4fdrKrV2BaKf8E81iVSgewEg-oi6LwGOJTNO1v92BAGQnkfNYAKYCU01FPEtvgyZcMI4Joi1C0Ae9ktr64n6ryjbYXHhxY1j1iaT3ckHVwnrcfFSmEFlVgr29XlB0PxPDfMm9aiXRABqtNDfkwDVeazbkUO8OoiWQAf0tSvJODFn8mCC6-PqCgOcetDb8-ImAnbIeBhkGqwGqN5hg3zZpWllwh0KOE_xVp1dpyi8Zy; 1P_JAR=2021-09-14-17; SEARCH_SAMESITE=CgQIwJMB; SID=BwgjyjBUve_nrojZokvP3r6FHvA5xKsDMYXmX4BI6VoTUC5ujOLG85oAltgP8LNID27DVg.; __Secure-3PSID=BwgjyjBUve_nrojZokvP3r6FHvA5xKsDMYXmX4BI6VoTUC5ueSysvQlC7n5-I8e7sdgIig.; HSID=APVnW9KcDk8gNWUFL; SSID=Avyffy8ZumJDYqoGm; APISID=PT1zU0pHCLTLIS9-/AynvPPljqwt6AChc9; SAPISID=FX2tPa-b3U-S4C9i/AS-F0GoA8ln4t7ypV; __Secure-3PAPISID=FX2tPa-b3U-S4C9i/AS-F0GoA8ln4t7ypV; __Secure-3PSIDCC=AJi4QfENxgiX7WVxnHwg7zF7v19mPy04cuEfo_0BDqCLDztGz_Zl-sv2U8GWdi0dYeScg6ky-pQ; SIDCC=AJi4QfF0JJonMObvuhYVBabtdREyJuCdAZUw817FnKfM6U8Vqv8moCjqws7Oc5lPp7eDjFQOupVh; OGPC=19023890-1:19024399-1:19022622-1:19025322-1:19022591-1:; __Secure-1PSID=BwgjyjBUve_nrojZokvP3r6FHvA5xKsDMYXmX4BI6VoTUC5uDBDUPwouSH8VZPyw49Sbkg.; __Secure-1PAPISID=FX2tPa-b3U-S4C9i/AS-F0GoA8ln4t7ypV; __Secure-1PSIDCC=AJi4QfHo9arSZvXX_IoXVw9cyKoOrEEcSwaGYJ3Wn1PCgDrB46LbdmUwJJn2Sc-c9FbTvDDiWkOb; OTZ=6125659_80_84_104220_80_446880; S=; DV=41W3wr1nebGA8J9VYJ1QRITG5NlUvlc-vjP4MDxhrAIAAADRB9mqW1MG8QAAAPwmflGDrSmKPAAAANb86siJVFT6haQBwFTHsPNRSWijI2kA0AsJxy-9qjoZSRoAlH__0KsWzq5SkgYA
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
TE: Trailers
```

* This data must be translated into binary (as does ALL data for computers).
* With HTTP/1.x this data is simply ASCII encoded text.
* HTTP/2 and HTTP/3 have a very specific binary format for transmitting data (more on this later!)
* This binary data (the HTTP request) is then "wrapped" or "enveloped" in a TCP packet.
    * Which means more literally, the TCP header values are prepended to the binary HTTP data we just created.
    * Even more specifically, the following values are filled out in binary, and prepended to the HTTP request:

![](https://www.lifewire.com/thmb/uOXb1Mr4IVWF2H8FBYl5PbC1JAo=/774x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/tcp-headers-f2c0881ea4c94e919794b7c0677ab90a.jpg)

* Briefly describe these values...
    * Ports are for multiplexing data between applications on a single host.
        * 80 is commonly used for HTTP, 224 for HTTPS
        * 53 for DNS
        * There are many more "well known" port numbers, but any port can theoretically be used for any application.
        * These are just numbers used by software to lookup the proper application (e.g. in a hash table), there is no physical analog for any given port.
    * Sequence and acknowledgment numbers are used to ensure reliable transport.
        * When more data than fits in one "packet" is sent these values identify which packets go in which order.
    * data offset says how many of the "optional data" is used, and therefore indicates where the "payload" (our HTTP request) starts.
    * The window size is related to TCP's congestion and flow control mechanisms. (more on that later!)
    * The checksum is a simple computed value used to check that data wasn't corrupted in transit.

* THIS data then needs to be wrapped in an IP header, so that it can be routed...

![](https://cdn.networkacademy.io/sites/default/files/inline-images/comparing-ipv4-and-ipv6-headers.png)

* Depending on which version of IP is being used, a different header is used. 
* The header itself indicates the version of IP being used, so that parsing programs can behave appropriately!
* BRIEFLY go over some of the values
    * traffic class is for enabling high and low priority transmissions.
    * flow label is an identifier to help distinguish between different streams of data from the same two computers (slightly conflated with Transport layer responsibilities!)
    * Payload length is the size of the TCP packet (which is itself the size of TCP headers plus the HTTP request)
    * Next Header is a field that indicates which type of data is in the payload. In our case, the payload of the IP packet is a TCP packet, so this field would be `6` which indicates TCP. There are many such values which have highly specific meaning and help to glue the layers together.
    * The addresses are our IP addresses.

* FINALLY, before we can send the data off our computer, we need to utilize whichever specific data-link protocol is appropriate. Lets assume the first hop from our laptop is an ethernet cable to our modem, then we'd wrap all this data in an Ethernet Frame:
    * (There are a few different kinds of Ethernet frame as the technology has evolved... here is one for Ethernet Type II)

![](https://en.wikipedia.org/wiki/File:Ethernet_Type_II_Frame_format.svg)

* The MAC addresses represent the two physical devices (our computers ethernet card, and the models ethernet card).
* Ethertype indicates which type of ethernet frame this is, Type II is the most common but there are others.
* The data is ALLLLLLLLLL the data from the HTTP, TCP, and IP layers.
* The CRC checksum is a mechanism to check that data has not been corrupted in transit.
* When this frame is transmitted to the modem, it must make a new frame (potentially using a different link technology) to send it along it's next hop. 
    * For example, maybe it's a fiber connection to a router owned by your ISP sitting on a telephone pole. The MAC addresses have to be changed, and the format of the data may change as well depending on the protocol used by the fiber optic cards on both devices.