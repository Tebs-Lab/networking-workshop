# Networking Workshop

This workshop targets students hoping to learn more about the details of computer networking. Students may or may not be programmers, but should have some fundamental IT knowledge. Ideally, all students would be comfortable using command line tools, but that will not be necessary to attend and benefit from the course.

## Course Outline

### 1. Network Layering Model

The Open Systems Interconnection (OSI) 7-layer model of networking is an extremely useful mental model for understanding, implementing, and debugging network issues. It is somewhat outdated in practice, having been largely replaced with the simplified 4-layer model popularized by TCP/IP. However, as modern networking protocols and tactics continue to evolve, the lines between different "layers" have been somewhat blurred. The OSI model gives names to distinct concerns that TCP/IP bundles unceremoniously as "application" level concerns.

**We'll learn...**

* What it means for networked applications to be "layered."
* Why this layering approach is (hugely) beneficial.
* What this layering means in terms of the data being transmitted over the wire.
    * In exquisite and excruciating detail.

### 2. The Application & Presentation Layers

The application layer is incredibly broad, especially in the TCP/IP version of the stack. We'll briefly examine these broad general guidelines for what might be done in the application layer, as well as the blurry line between what is "application" and what is "presentation." Then, we'll dive into two of the most widely used application layer protocols: DNS and HTTP

**We'll learn...**

* What the responsibilities of the application layer are, broadly speaking.
* The specific purpose, function, and details of HTTP.
* The specific purpose, function, and details of DNS.
* The lifecycle of a web request with respect to HTTP and DNS, from the perspective of a web browser.

### 3. The Session and Transport Layers

The session and transport layers (combined into one layer in TCP/IP) are responsible for maintaining a network session, and coordinating communication at a "host-to-host" level. These layers are aware that two computers are communicating over a network, but are agnostic to/unaware of the specific mechanisms connecting those two computers. Protocols at this layer might (or might not) provide connection management, reliable data delivery, flow control, and connection multiplexing.

**We'll learn...**

* What the responsibilities of the transport layer are, broadly speaking.
* The details of the simplest transport layer protocol: UDP.
* The specific purpose, function, and some details of TCP including:
    * Connection establishment and management.
    * Control flow behavior.
* The specific purpose, function, and some details of SSL/TLS, including:
    * The role of "Certificate Authorities"
    * The details of the key-generation process.

### 4. The Network Layer

The network layer is responsible for getting data from point A to point B. We'll learn about the breakaway star of the networking layer: the Internet Protocol (IP). To understand how IP works in the real world, we'll also have to understand the role of a critical supporting protocol called BGP and the concept of Autonomous Systems.

**We'll learn...**

* The purpose, function, and some details of the Internet Protocol, including:
    * Some of the differences between IPv4 and IPv6.
    * Subnets and CIDR notation.
* The purpose, function, and some details about the Border Gateway Protocol.
    * "Routing" vs "Forwarding"
* The purpose and function of two other supporting protocols:
    * Network Address Translation (NAT)
    * Dynamic Host Configuration Protocol (DHCP)

### 5. The Link and Physical Layers

These two layers are tightly linked (and in fact are combined in the TCP/IP stack). They relate to the hardware that physically sends information across the medium of choice. Ethernet, Wi-Fi, and 4G/5G are the most well known such protocols. These layers are of critical concern to folks building networking hardware, and tend to be less important to software developers who assume this stuff "just works" and write their software accordingly.

**We'll learn...**

* A little bit about the physical transmission mechanism of Ethernet.
* Enough about how Wi-Fi works to wonder how it is even possible.
* The basics of the Address Resolution Protocol (ARP).