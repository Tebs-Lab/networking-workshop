# DNS

In this section we describe the purpose and function of DNS.

## The Basics

* The Domain Name Service (DNS) is a distributed key-value database mapping Uniform Resource Locators (URLs) to Internet Protocol (IP) addresses.
* It is both hierarchical and distributed. 
* There are 3 types of DNS servers:
    * Root nameservers
    * Top Level Domain nameservers
    * Authoritative nameservers
    * Combined, these 3 types of servers contain the mappings of URLs to IP addresses.
    * They are arranged in a hierarchy: root -> TLD -> authoritative. 
    * Each member of the hierarchy has a different set of responsibilities related to maintaining the mapping of URLs to IP addresses.
    * We will describe their individual responsibilities in more detail momentarily. 

* In addition to the 3 server types that comprise the actual DNS distributed database, there are servers called DNS resolvers which are designed with knowledge of how DNS works specifically to query the other three types of servers.
    * Unlike the nameservers, resolvers are not the "source of truth" rather they query the "sources of truth" and only store the answers temporarily (caching).

## The Nameserver Hierarchy

* Root nameservers are only responsible for knowing the answer questions of the following form:"Which TLD server should I ask if I am looking for www.google.com?"
    * These root servers look at the "top level domain" (com in this case) in order to answer that question.
    * There are 13 root nameserver addresses i.e. there are 13 IP addresses reserved for DNS nameservers.
    * At one point it was literally true that there were just 13 root nameservers.
    * Today there are over 600, and we use something called "anycast routing" to allow these globally distributed servers to "share" an IP address.
    * You can see the addresses, and who operates them here: https://www.iana.org/domains/root/servers

* Top Level Domain (TLD) servers are only responsible for knowing the answer to questions in the form: "Which Authoritative nameserver should I ask if I am looking for www.google.com"
    * The last portion of the URL (.com, .gov, .co.uk, .jp, and so on) is called the "top level domain" and each TLD maintains it's mapping of domains to authoritative nameservers separately. 
    * ICANN's IANA branch manages a list of all such TLD servers.
    * The TLD servers themselves are operated by a wide range of groups, including governments and NGOs.
    * You can see a list of who manages each TLD here: https://www.iana.org/domains/root/db

* Authoritative name servers actually know the answer questions in the form of: "What is the IP address for www.google.com?"
    * Authoritative nameservers are often operated by the company in question, e.g. Google manages their own authoritative nameservers.
    * Equally as often that work is outsourced, many "registrars" (such as GoDaddy and NameCheap) offer Authoritative Nameservers to their customers for a fee, or included in the price of the domain name itself.

* Finally, DNS resolvers know how to properly query DNS servers in order to find the IP address for a URL.
    * Most ISPs provide a DNS resolver for their customers.
    * Cloudflare operates a DNS resolver at 1.1.1.1
    * Google operates a DNS resolver at 8.8.8.8
    * These resolvers are NOT part of the database, rather they are simply tools provided to query the database.
    * Semi-centralized Resolvers do take some pressure off the DNS system as a whole, though, for example by providing caching services that reduce the number of queries that actually make it to one of the DNS nameservers.

### An Example

Suppose you typed www.google.com into your browsers URL bar and hit enter. Further suppose your computer's DNS resolver is pointing to Cloudflare's 1.1.1.1 resolver, and further assume that all of the relevant cache entries on all relevant servers are empty right now.

* **Diagram this on the blackboard**
* First your browser checks it's cache (not there)
* Your browser tells your OS it wants to make a DNS query (although it is theoretically possible for the application to bypass your system DNS settings, it's not super common to do so).
* Your OS looks in it's DNS cache.
* Your OS looks up which DNS resolver you have configured the system to use (by default this is often set by a DHCP server, which more or less means it'll initially point to an IP address provided by your ISP, so probably your ISPs resolver.)
* Your OS makes a query to that resolver, asking for the IP address.
* The resolver receives the query, and first asks one of the root servers "which TLD server do I need?"
* The root server will respond with the names and IP addresses of the appropriate TLD server.
* The resolver will then ask that TLD server, "which server do I need?"
* The TLD server will respond with the names and IP addresses of the appropriate authoritative servers.
* The resolver will ask one of those authoritative servers "whats the IP address?"
* The authoritative server will tell it what the address is.
* The resolver will send that address back to you.
* Your OS will forward that data do your browser.
* Your browser will begin the process of opening a TCP connection for that IP address. Yay!

### Types of Queries

* DNS does kind of a lot of things... but the most common record types are:
* A and AAAA records, these records are for mapping URLs to IP addresses. A is for IPv4, AAAA is IPv6
* NS records, these are for mapping URLs to their appropriate nameserver, rather than the IP address for the actual service.
* CNAME records forward one domain to another domain (like an alias) and do NOT involve IP addresses at all.
* MX records are like A records, but for email servers specifically.
* For more types of records see: https://www.cloudflare.com/learning/dns/dns-records/

### Record Format

* This is a great resource: https://routley.io/posts/hand-writing-dns-messages/
* Record format stuff comes from the RFCs: https://datatracker.ietf.org/doc/html/rfc1035
* Every DNS messages have the same format, with 5 sections:

```
+---------------------+
|        Header       |
+---------------------+
|       Question      | the question for the name server
+---------------------+
|        Answer       | Resource Records (RRs) answering the question
+---------------------+
|      Authority      | RRs pointing toward an authority
+---------------------+
|      Additional     | RRs holding additional information
+---------------------+
```

* The contents of the header are below.
* Questions are queries.
* Answer is for records that directly answer the question being asked.
* Records in Authority say "I don't know the answer, but you should ask this server instead."
* Records in Additional are helpful details, for example if a server doesn't have the A record you asked for, but knows the server that does, the "authority" section might be an NS record with the name of the proper server to ask next, and the "additional" section might be an A record for that next server (meaning you get the name and IP address, thanks!)

```
0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                      ID                       |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|QR|   Opcode  |AA|TC|RD|RA|   Z    |   RCODE   |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    QDCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    ANCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    NSCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    ARCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

* ID is an identifier, pretty much arbitrary, but a response to a query should use the same ID as the query itself.
* QR is a single bit ("Query/Response"), 0 if query, 1 if response.
* Opcode specifies the type of query, there are 3: 
    * 0: Standard query 
    * 1: Inverse query
    * 2: Server status request
* TC indicates if the message has been truncated (most queries won't be, but sometimes a DNS query is too long to fit in one packet)
* RD requests "recursion" (i.e. asks that a response not be returned unless it is actually the answer to the question we asked), but many DNS nameservers just ignore this, whereas resolvers usually respect it. 
* The remaining "count" values are all just "how many records of each type are in this query/response."
    * In a single query only the first QDCOUNT has 1, and the rest have 0. 
    * In a response to an NS request indicating that there are 4 authoritative nameservers the ANCOUNT would be 4.


* The contents of the Question section are:

```
0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                                               |
/                     QNAME                     /
/                                               /
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     QTYPE                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     QCLASS                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

* QNAME is the URL in question.
* QTYPE is an integer that maps to one of the record times (A records are type 1, for example.)
* QCLASS is a DNS class (see the list of classes here: https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml)
    * But, in practice, this is basically always 1 (for "Internet")

* The contents of each other section are "records" (as in an A record, an AAAA record, an NS record...)

```
0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                                               |
/                                               /
/                      NAME                     /
|                                               |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                      TYPE                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     CLASS                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                      TTL                      |
|                                               |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                   RDLENGTH                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--|
/                     RDATA                     /
/                                               /
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

* Name is the URL again.
* Type is the type of record (A, NS, AAAA...)
* Class is the same as QCLASS (e.g. nearly always 1 for "internet")
* TTL is "time to live" which is how long your'e allowed to cache this record (in seconds).
* RDLENGTH indicates the length of the RDATA section.
* RDATA contains the proper value for the record type (in an A record, it's an IP address. In an NS record, it's another URL...)

* Lets do the exercise!