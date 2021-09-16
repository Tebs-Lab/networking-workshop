# Internet Protocol, Border Gateway Protocol, and "Routing" vs "Forwarding"

## Addressing

* Perhaps the key assumption underlying the Internet is that every accessible device needs an address.
* Originally, these addresses were mainly meant to be unique per every device... 
    * but now we have some ways in which this assumption is violated.
    * Anycast addressing
    * Local Area Networks
    * ...

* Still, if a device wishes to receive data from other computers in the internet, those computers need to know "where" to send it.
* You *can* send data into the internet without a valid IP address, but you can't expect a reply.
* IPv4 Addresses are 32 bits long, and are typically displayed as 4 8-bit chunks (e.g. 123.45.67.89)
* **Micro-exercise: Your IP Address?**
    * First, google "whats my ip", write down what Google tells you.
    * Now, in the terminal, use one of the following commands: `ipconfig getifaddr en1` for wireless, or `ipconfig getifaddr en0`
    * Why didn't these values match?
    * **The answer is that one is your IP address on the local area network, and the other is your public IP address**
        * More on why this is done in a minute!!

## Getting an Address

* IP addresses are managed by Internet Assigned Numbers Authority (IANA), which has overall responsibility for the Internet Protocol (IP) address pool, and by the Regional Internet Registries (RIRs) to which IANA distributes large blocks of addresses.
* There are 5 RiRs, which broadly govern large geographic areas.
* These organizations generally allocate IP addresses in "blocks" to people who apply.
    * In the early days this process was somewhat less formal and some organizations were allocated very large.
    * As it became clear that there were going to be many more internet connected devices than possible IP addresses (2^32 == 4,294,967,296, "only" 4.3 billion) the RiRs became somewhat more stingy.
    * IPv6 solved the scarcity issue, so it is once again possible to get fairly large chunks of addresses.
        * IPv6 has 128 bit addresses == 2^128 valid addresses ~= 3.4*10^34 (that's a number with 34 0's on the end of it, which is a lot.)

* However, the vast majority of internet users never actually go through this process to get their own IP addresses.
* Instead, ISPs apply for very large blocks of addresses, and assign them to their customers' home modem.
* Then, those customers use a "Local Area Network" and a protocol called "Network Address Translation" to enable a situation where everyone on the local area network can "share" a public IP address (actually the address of the modem).
    * The Dynamic Host Configuration Protocol (DHCP) is used BOTH by the ISP to assign an IP address to the customer AND by the routers on the LAN to assign IP addresses within the local area network to each of those devices. (more on this in a moment)
    * NAT provides multiplexing to the individual devices on the LAN (more on that in a moment)

## CIDR

* "blocks" of IP addresses are generally communicated in Classless inter-domain routing (CIDR) notation
* This notation consists of the address, and the number of bits within that address that are fixed.
* For example, the address 192.168.1.1/32 means all 32 bits are "fixed" and this CIDR address identifies exactly 1 host.
    * The address 192.168.0.0/16 means the top 16 bits are fixed (the ones associated with 192.168), and the bottom 16 bits could be anything.
        * Therefore 192.168.0.0/16 means "any IP address starting with 192.168"
        * 192.168.1.0/24 means "any address starting with 192.168.1"

* Routing and forwarding are often done on the basis of blocks, rather than by individual IP addresses.
    * This is to save memory!!
    * Imagine if every router needed to save all 4B IPv4 addresses in it's forwarding table, that's untenable!
    * Instead, since IP addresses are often assigned in such blocks, a router can just look at the top few bits and know where to send the data.

* This is also where the term "subnet mask" comes from.
    * In a /32 CIDR address, the subnet mask is 255.255.255.255 (all 1's in binary)
    * In a /24 CIDR address, the subnet mask is 255.255.255.0 (11111111 11111111 11111111 00000000)
    * The "mask" operation is a binary AND operation, and we "mask off" the bits where the "mask" is a zero.
    * The address 192.168.10.1/24 is masked by 255.255.255.0 and results in 192.168.10.0
    * This is useful in the routing context, we "mask" certain subnets because the same outbound wire is the shortest path to ANY of the devices on the subnet identified by 192.168.10.x

* CIDR notation can use any value 0-32 on IPv4 and any value 0-128 on IPv6, but you'll find that some values are more common than others (e.g. /24, /16, /8)

## DHCP

* DHCP is a protocol designed to allocate IP addresses to devices.
* A DHCP server is provided with a pool of IP addresses
    * In the context of an ISP this pool is some set of addresses owned by the ISP.
    * Often, there will be separate DHCP servers for separate geographic regions, and the ISP's address pool will be divided amongst those servers.

* IP addresses from the pool are "leased" by the DHCP server to specific devices.
    * The server tracks which address are currently leased, as well as information about the device to which it is leased (e.g. the MAC address)

* In the context of an ISP...
    * leasing an address to a customer, these leases are typically long, and often the same address will be reassigned to an individual customer when the lease is up.
    * This process happens once during the initial setup of your home router, then a renewal process happens semi-regularly.

* In the context of a home or local area network, DCHP happens when you login to the WiFi access point or when you plug in an ethernet cable to the device.
    * In this context leases are typically shorter.
    * Furthermore, the address pool in this context will (very likely) be one of the address blocks reserved for use by local area networks (more on this in a moment)

* Devices in this context have to communicate with each other WITHOUT IP ADDRESSES!! :O
    * This is generally done using the "data-link" layer. Here's the idea:
    * First, the client sends a "DHCP Discover" message to the "broadcast MAC address (ff:ff:ff:ff:ff)
        * Messages sent to this address are forwarded to all available physical devices.
            * Some devices may not play nice with this rule, but the vast majority do.
        * Eventually, the DHCP Discover message is forwarded to a DHCP server
        * The DHCP server responds with a DHCP Offer message, which includes an IP address, a lease duration, and some other details.
            * The DCHP server knows the MAC address to send this to, because that was included in the DCHP Discover request.
            * It's possible for the client to receive more than one offer this way.
        * The client responds to the Offer with a DHCP Request (which is more like accepting the offer than requesting something...) and the DHCP server officially assigns the IP address to the device.
            * This will cause forwarding tables to be updated on the ISPs network (more on that in a moment!)
        * Finally, the server sends an acknowledgment indicating that the IP has been assigned and reserved. 
        * This process is fundamentally the same for the ISP or for the LAN.

## Local Area Networks & Network Address Translation

* The creators of the internet anticipated the desire for local networks that used the same set of protocols, without the need to be connected to any other networks. 
* And, as it became clear in the early 2000s that we would run out of IPv4 addresses, it became necessary for groups of computers to share a public IP address.

* The solution to both problems is NAT!

* several blocks of IP addresses are reserved for use by LANs, which means that no computer can have a public IP in that block.
* Two such blocks are 192.168.0.0/16 and 10.0.0.0/24
    * There is one more in IPv4 and another in IPv6

* Network Address Translation allows every device on the local network to use an address from one of these blocks, while exposing only 1 IP address to the public.
    * The router then becomes responsible for tracking all the requests coming out of any Local IP addresses, and forwarding traffic from the broader internet properly to it's intended host on the local network.

* There are a handful of implementations of NAT, but here is the basic idea:
    * Host sends a message to the internet, say the local address is 10.0.1.3:7777
    * The gateway (the router/modem with the public IP address) makes a note of that IP/port combo, and assigns one of it's own  port numbers to that combo. Say the gateway's public IP is 123.4.5.6, and it chose port 7890.
    * Now, when the other end of that connection responds, it sends a message bound for 123.4.5.6:7890
        * The gateway looks up that port (7890) in it's NAT table, and forwards it to the local network to 10.0.1.3:7777

## Routing vs Forwarding

* In networking you'll hear people use the terms "routing" and "forwarding" in very specific ways.
    * Routing refers to the process of deciding which route between point A and point B should be used. 
        * In internet routing this process involves both "intra" network routing and "inter" network routing.
        * Graph theory and those shortest path algorithms from college play a major role.
        * Conditions on the network are constantly in flux, which means routing decisions may need to adapt in real time.
        * All of the "routing" work is done ahead of time, meaning routers have already decided what route any particular IP address should be sent along BEFORE that packet arrives.
        * This information is saved in something called a "forwarding table"
    
    * Forwarding refers to the comparatively simple process that occurs when a packet arrives at a router...
        * Look up the address in your forwarding table (typically masked to some subnet).
        * Send it out on the interface designated in the forwarding table.

* Forwarding tables might be updated in real-time (or they might be set by hand!)
* Regardless though, a shortest path is NOT computed in real time for every packet that arrives at a router.
    * Instead, the "routing" work is done asynchronously, and when updates happen packets arriving after the update are routed according to the new rules.

## Autonomous Systems, BGP 

* The internet is a collection of networks, and all of those networks are allowed to manage themselves fairly independently. 
    * They can use different routing schemes and any kind of hardware they want, for example.
    * Interior Gateway Protocols (there are several) are used to perform "itra" network routing (routing contained to one network)
        * Open Shortest Path First (OSPF) and Routing Information Protocol (RIP) are two examples.

* However, these autonomous systems agree to follow some rules to allow for "inter-network" routing, specifically these rules are encoded in a protocol called the Border Gateway Protocol. Currently on version 4: BGP4
    * "Border Gateway" because the nodes that really matter for this protocol are nodes on the "border" of a network, nodes that have connectivity to other networks (hence "gateway")

* BGP is a very complex protocol with a lot of rules and edge cases, but the fundamentals aren't too hard to understand...
* There are 3 important tables tracked by BGP nodes, and which govern their routing behavior.

* Firstly, know that all of the Border Gateway nodes must have IP addresses, and typically use TCP to communicate with each other.
    * This implies that nodes must be connected, but not *directly* — there can be some infrastructure in between two BGP nodes.

* BGP relies on three tables to run properly
    * The "neighbor table" tracks which other BGP nodes are reachable from this network.
        * These neighbors communicate regularly about which subnets are reachable from their network.
        * These nodes are NOT necessarily the actual routers at the edge of the network, but rather they are the nodes coordinating the BGP information amongst the network peers.
        * These nodes inform each other with information that will end up in the "BGP table"

    * The "BGP Table" (also called BGP topology table and BGP Routing Information Base (RIB)) 
        * This contains information about actual routing, e.g. IP addresses of all the gateways in the neighboring networks, and reachability information for all the subnets that can be reached from within that network.
        * This table will generally contain MANY possible routes to any given IP address and/or subnet.

    * The BGP Routing Table contains only the "best" (shortest) paths from THIS autonomous system to any given IP address or subnet.

* BGP routes are built by a system of announcements.
    * BGP nodes announce all the IP Subnets that they themselves own (e.g. a node operated by Comcast might announce all the subnets owned by Comcast), and broadcast this to their BGP neighbors.
    * These announcements are forwarded by the neighbors that receive it, and they add their Autonomous System Number to the AS_PATH attribute on the announcement. 
    * As these announcements cascade through the system, BGP nodes add them to their BGP table.
    * If one of these announcements contains either a NEW subnet (from the AS perspective) or a path with fewer AS's in the AS_PATH to a known subnet, then the AS will update their BGP Routing table.

* Route Withdrawals work similarly...
    * A BGP node announces that it no longer has a particular route.
    * Other nodes remove that route from their BGP RIB
    * If that route was the best route, they have to find an alternative in their BGP RIB, or themselves announce that they can no longer reach that particular subnet.

## IPv6

* The most important (but far from only) difference between IPv4 and IPv6 is that V6 has 128-bit addresses rather than v4 addresses.
    * On a binary level (bitwise) addressing, subnetting, and masking work exactly the same way.
    * However, we also display IPv6 addresses for humans in a different format using hexadecimal.
        * e.g.: `fe80::18d2:4106:b4f0:33dc`
        * Colons are the new dots, to break up sections.
        * Each colon separated section is 4 characters => 16 bits.
        * double colons indicate a section of all 0's, so in our address above we have 5 represented 16-bit sections
            * thats only 80 of the 128 bits, therefore in-between those colons there are 48 bits all with a 0 value.
        * CIDR rules still apply, so you can represent a subnet as `fe80::18d2:4106:b4f0::/112`
            * Although the preferred nomenclature is now "prefix length" rather than "subnet mask"
            * IPv6 also has a strong preference for prefix lengths that are a multiple of 4
            * Not also you'll typically see subnets ending with the double colon syntax.

* In theory, IPv6 eliminates the need for NAT (which was mostly a stop gap measure to save IP addresses)
    * But in practice NAT remains prevalent, in part because of momentum, in part because IPv4 addresses are still commonly used, and in part because NAT provides a sort of de-facto firewall that provides some small degree of protection from people outside the local network.
        * Although having NAT is certainly NOT a good reason to not use an actual firewall.

* IPv6 no longer supports broadcast which IPv4 did support, but ipv6 does support anycast (which IPv4 did not support directly, but was emulated through what could be considered 'clever hacks')