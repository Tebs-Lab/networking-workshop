# Explore DNS With Dig

In this exercise you'll learn about DNS by using `dig` to make DNS queries. These are the primary objectives of this exercise:

* Expose the different responsibilities of different members of the DNS hierarchy.
* Explore how different DNS servers respond differently to different DNS queries.
* Familiarize yourself with the different kinds of DNS record types, and distinguish between them.

`dig` is a command line tool that comes installed on most unix and linux systems, or can be installed with your favorite package manager. If not, you can use [this web interface](https://www.subnetonline.com/pages/network-tools/online-dig.php) -- but the user experience is much worse so I strongly suggest you use the command line instead.

## IP Addresses and Name Servers

There are many different kinds of DNS records. The two most important types of record are:

* A and AAAA -- A and quad A records map domain names to IP addresses. A records are for IPv4 addresses and AAAA records are for IPv6 addresses.
* NS -- NS records map domain names to *other* domain names. Specifically they map a domain name to the name of it's authoritative DNS server.

Lets use `dig` to explore the differences between these record types. Try the following in your terminal:

```
dig A google.com
```

My response looks like this, yours ought to look similar:

```
; <<>> DiG 9.10.6 <<>> A google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39086
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1452
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		68	IN	A	172.217.2.238

;; Query time: 13 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Fri Jun 22 15:05:03 PDT 2018
;; MSG SIZE  rcvd: 55
```

**What IP address did your DNS resolver tell you to use for google.com?**
> Your answer here.

**You may have a different IP address than I received. Is this expected behavior for DNS? Why or why not?**
> Your answer here.

**If you did get a different IP address, how different is it? Can you spot any similarities to my result?**
> Your answer here.

**What happens if you paste the IP address you received into your web browser's URL bar?**
> Your answer here.

**This line (found near the bottom of the output from `dig`) is probably different for you:**

`;; SERVER: 1.1.1.1#53(1.1.1.1)`.

**1.1.1.1 is the IP address of the DNS resolver I am using -- a service provided by CloudFlare.**

**Use the `whois` terminal command to determine who is operating your DNS resolver. For example, `whois 1.1.1.1` will tell that 1.1.1.1 is an "APNIC and Cloudflare DNS Resolver project"**
> Who operates your DNS resolver?

**Now try this:**

```
dig AAAA google.com
```

**What is the difference between the answer to this query, and the last query?**
> Your answer here.

**Now try this:**

```
dig NS google.com
```

**My results look like this, and yours should look similar:**

```
; <<>> DiG 9.10.6 <<>> ns google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59422
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1452
;; QUESTION SECTION:
;google.com.			IN	NS

;; ANSWER SECTION:
google.com.		6259	IN	NS	ns1.google.com.
google.com.		6259	IN	NS	ns2.google.com.
google.com.		6259	IN	NS	ns3.google.com.
google.com.		6259	IN	NS	ns4.google.com.

;; Query time: 15 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Fri Jun 22 15:15:27 PDT 2018
;; MSG SIZE  rcvd: 111
```

**4 servers are identified by name, ns1.google.com, ns2.google.com, ns3.google.com and ns4.google.com.**

**What are these servers?**
> Your answer here.

**How can you use `dig` to determine an IP address for each of these servers?**
> Your answer here.

**Use `whois` to determine who owns those IP addresses?**
> Your answer here.

In this exercise, we have been relying exclusively on your "default DNS resolver" to answer questions for us. In the [next exercise](https://gist.github.com/tebba-von-mathenstein/ff963772b6f770f8cb41bfe05b38721e) we will use `dig` to explore the DNS hierarchy by sending DNS queries directly to root, TLD, and authoritative servers.

An answer key for this exercise [can be found here](https://gist.github.com/tebba-von-mathenstein/4a85da23e60159829cb8517b2c8b32b8)

## Querying Specific Servers

**Make the following request with `dig`**

```
dig a ns1.google.com
```

This command returns the IP address of one of Google's authoritative namer servers. My results look like this:

```
; <<>> DiG 9.10.6 <<>> a ns1.google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33256
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1452
;; QUESTION SECTION:
;ns1.google.com.			IN	A

;; ANSWER SECTION:
ns1.google.com.		5209	IN	A	216.239.32.10

;; Query time: 14 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Wed Jul 25 11:16:43 PDT 2018
;; MSG SIZE  rcvd: 59
```

Your default DNS resolver (mine is CloudFlare's 1.1.1.1) responded to our request. But we can use `dig` to send messages directly to other DNS servers.

**Now make the following two requests with `dig`. In the second request, replace 216.239.32.10 with whatever IP address you recieved in the last request.**

```
dig a google.com
dig @216.239.32.10 google.com
```

**Did you get the same IP address for both requests?**
> Your answer here

**If so, why might this be? If not, why not?**
> Your answer here

**If you got two different IP addresses, try using `traceroute` to determine if one of those two addresses is "closer" to you in terms of number of nodes on the internet. If you did not get two different IP addresses, try querying another one of Google's authoritative servers to get a new IP address for google.com**

**For example, I have received these IP address for google.com so far:**

`216.58.194.174` and `172.217.5.110` so I am going to run:

```
traceroute 216.58.194.174
traceroute 172.217.5.110
```

For me, the first IP address took 15 hops and the second took only 9. `172.217.5.110` is topologically closer to me on the internet. Which IP address was closer to you?

## Exploring The DNS Hierarchy

The DNS Hierarchy has 3 classes of servers. Root -> TLD -> Authoritative.

Root servers know about TLD servers, TLD servers know about Authoritative servers, and Authoritative servers know about specific websites and web services. In this part of the exercise we'll explore the process of resolving a DNS query the way a DNS resolver would if it didn't have any information in it's cache.

**First, we will use your local resolver to find the IP address of two root server. Try the following:**

```
dig A a.root-servers.net
dig A b.root-servers.net
```

**What IP addresses identify these two root servers?**

> Your answer here

All the root servers have similar names, you can query for a-m.root-servers.net. These root servers have fixed IP address that do not change -- this is critical to the Internet's infrastructure. If the root IP address changed... who would we ask to determine the IP address of those servers?

**Try using `traceroute` to determine which of those two root servers is closer to you**

> What traceroute commands did you run? Which server is closer?

**Using the IP address of one of the root servers make a query directly to a root server asking for an A record for google.com.**
> What dig command should you use?

Your result should not be anything like the results from the previous section -- here's what I got:

```
; <<>> DiG 9.10.6 <<>> A a.root-servers.net
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12648
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1452
;; QUESTION SECTION:
;a.root-servers.net.		IN	A

;; ANSWER SECTION:
a.root-servers.net.	632	IN	A	198.41.0.4

;; Query time: 14 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Fri Jun 22 15:32:43 PDT 2018
;; MSG SIZE  rcvd: 63

Tylers-MacBook-Pro:dns-exercises tylerbettilyon$ dig @198.41.0.4 A google.com

; <<>> DiG 9.10.6 <<>> @198.41.0.4 A google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8001
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 13, ADDITIONAL: 27
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1472
;; QUESTION SECTION:
;google.com.			IN	A

;; AUTHORITY SECTION:
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.

;; ADDITIONAL SECTION:
a.gtld-servers.net.	172800	IN	A	192.5.6.30
b.gtld-servers.net.	172800	IN	A	192.33.14.30
c.gtld-servers.net.	172800	IN	A	192.26.92.30
d.gtld-servers.net.	172800	IN	A	192.31.80.30
e.gtld-servers.net.	172800	IN	A	192.12.94.30
f.gtld-servers.net.	172800	IN	A	192.35.51.30
g.gtld-servers.net.	172800	IN	A	192.42.93.30
h.gtld-servers.net.	172800	IN	A	192.54.112.30
i.gtld-servers.net.	172800	IN	A	192.43.172.30
j.gtld-servers.net.	172800	IN	A	192.48.79.30
k.gtld-servers.net.	172800	IN	A	192.52.178.30
l.gtld-servers.net.	172800	IN	A	192.41.162.30
m.gtld-servers.net.	172800	IN	A	192.55.83.30
a.gtld-servers.net.	172800	IN	AAAA	2001:503:a83e::2:30
b.gtld-servers.net.	172800	IN	AAAA	2001:503:231d::2:30
c.gtld-servers.net.	172800	IN	AAAA	2001:503:83eb::30
d.gtld-servers.net.	172800	IN	AAAA	2001:500:856e::30
e.gtld-servers.net.	172800	IN	AAAA	2001:502:1ca1::30
f.gtld-servers.net.	172800	IN	AAAA	2001:503:d414::30
g.gtld-servers.net.	172800	IN	AAAA	2001:503:eea3::30
h.gtld-servers.net.	172800	IN	AAAA	2001:502:8cc::30
i.gtld-servers.net.	172800	IN	AAAA	2001:503:39c1::30
j.gtld-servers.net.	172800	IN	AAAA	2001:502:7094::30
k.gtld-servers.net.	172800	IN	AAAA	2001:503:d2d::30
l.gtld-servers.net.	172800	IN	AAAA	2001:500:d937::30
m.gtld-servers.net.	172800	IN	AAAA	2001:501:b1f9::30

;; Query time: 30 msec
;; SERVER: 198.41.0.4#53(198.41.0.4)
;; WHEN: Fri Jun 22 15:34:10 PDT 2018
;; MSG SIZE  rcvd: 835
```

**What information has the the root server given us?**
> Your answer here

**Which class of server in the hierarchy are a-m.gtld-servers.net?**
> Your answer here

**Why didn't we get an A record for google.com like we asked for?**
> Your answer here

**How can we use this information to get closer to the answer we want (an A record for google.com)**
> Your answer here

**What is the relationship between the records in the Authority section and the Additional section?**
> Your answer here

**Ask one of these TLD servers the same question, for an A record for google.com.**
> What dig command did you use?

**If you did [part 1]() of this exercise the response should look familiar, this is what I got:**

```
; <<>> DiG 9.10.6 <<>> @192.55.83.30 A google.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51556
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 4, ADDITIONAL: 9
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.			IN	A

;; AUTHORITY SECTION:
google.com.		172800	IN	NS	ns2.google.com.
google.com.		172800	IN	NS	ns1.google.com.
google.com.		172800	IN	NS	ns3.google.com.
google.com.		172800	IN	NS	ns4.google.com.

;; ADDITIONAL SECTION:
ns2.google.com.		172800	IN	AAAA	2001:4860:4802:34::a
ns2.google.com.		172800	IN	A	216.239.34.10
ns1.google.com.		172800	IN	AAAA	2001:4860:4802:32::a
ns1.google.com.		172800	IN	A	216.239.32.10
ns3.google.com.		172800	IN	AAAA	2001:4860:4802:36::a
ns3.google.com.		172800	IN	A	216.239.36.10
ns4.google.com.		172800	IN	AAAA	2001:4860:4802:38::a
ns4.google.com.		172800	IN	A	216.239.38.10

;; Query time: 30 msec
;; SERVER: 192.55.83.30#53(192.55.83.30)
;; WHEN: Fri Jun 22 15:37:59 PDT 2018
;; MSG SIZE  rcvd: 287
```

**What did the TLD server tell us?**
> Your answer here

**Once again, why didn't we get an A record for google.com?**
> Your answer here

**How does this information help us get closer to an A record for google.com?**
> Your answer here