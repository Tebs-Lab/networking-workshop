# IP Addressing Exercise

## Part 1: Examining Routes

Traceroute is a program that can tell you which path (in terms of IP addresses). We're going to use `traceroute` and `dig` to examine some paths from our computer to other nodes in the network.

1. Use dig to find an IPv4 address for google.com.

```
$ dig google.com

; <<>> DiG 9.10.6 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59913
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		10	IN	A	142.250.189.174

;; Query time: 19 msec
;; SERVER: 192.168.86.1#53(192.168.86.1)
;; WHEN: Thu Sep 16 13:17:16 MDT 2021
;; MSG SIZE  rcvd: 55
```

2. Use traceroute to discover the the path from your computer to that ip address.

```
$ traceroute -a google.com
traceroute to google.com (216.58.194.206), 64 hops max, 52 byte packets
 1  [AS0] 192.168.86.1 (192.168.86.1)  1.258 ms  0.648 ms  0.537 ms
 2  * * *
 3  [AS16591] 23-255-224-134.mci.googlefiber.net (23.255.224.134)  3.388 ms  1.909 ms  1.981 ms
 4  * * *
 5  [AS16591] 23-255-225-53.googlefiber.net (23.255.225.53)  16.533 ms  16.321 ms  16.073 ms
 6  [AS15169] 72.14.202.73 (72.14.202.73)  16.589 ms  16.446 ms  16.479 ms
 7  * * *
 8  [AS15169] 72.14.235.2 (72.14.235.2)  16.402 ms
    [AS15169] 108.170.236.60 (108.170.236.60)  16.420 ms
    [AS15169] 72.14.235.2 (72.14.235.2)  16.450 ms
 9  [AS15169] 108.170.237.105 (108.170.237.105)  16.381 ms  16.823 ms
    [AS15169] 108.170.237.107 (108.170.237.107)  16.105 ms
10  [AS15169] sfo03s01-in-f14.1e100.net (216.58.194.206)  16.185 ms
    [AS15169] 142.250.234.141 (142.250.234.141)  17.052 ms
    [AS15169] sfo03s01-in-f14.1e100.net (216.58.194.206)  16.306 ms
```

3. Look carefully at the output and...
    * Identify how many hops it took to get to google.
        * **For me, it was 10 hops.**

    * Try and identify how many autonomous systems are represented.
        * Hint: use the `-a` option of traceroute to include AS numbers!
        * Bonus: use the `whois` command to find out who these IP's belong to!
        * **For me, as soon as I got out of my local network I was on Google's infrastructure. So I only traveled through ONE autonomous system, which makes sense because my ISP is Google Fiber**
    * Your output might sometimes include *'s, what does this mean?
        * **Traceroute uses a protocol called ICMP, and some routers don't respond to ICMP requests, even though they do forward your data. That's why these are *'s, those routers aren't responding to ICMP.**

    * Your output might sometimes include multiple IP addresses "per hop", what does this mean?
        * **This happened in my output on hops 8, 9, and 10. This means that during the three requests a different router responded for each one. This implies there are multiple paths from me to google.com, and that google has decided to multiplex my traffic through a different node at least once.**


## Part 2: Engineers Just Wanna Have Fun

* Execute the following command:

```
traceroute bad.horse
```

* This might take a minute or two to complete. In the meantime get some context for this question by watching this youtube video from Dr. Horrible's Sing-Along Blog: [https://www.youtube.com/watch?v=VNhhz1yYk2U](https://www.youtube.com/watch?v=VNhhz1yYk2U)

* Look carefully at the IP addresses, then try to decide how this outcome was achieved.

```
192.168.86.1 (192.168.86.1)  1.396 ms  0.643 ms  0.507 ms
 2  * * *
 3  23-255-224-134.mci.googlefiber.net (23.255.224.134)  2.112 ms  1.796 ms  2.003 ms
 4  * * *
 5  * * *
 6  23-255-224-29.mci.googlefiber.net (23.255.224.29)  25.672 ms  23.577 ms  23.250 ms
 7  port-channel3.core3.lax2.he.net (184.104.197.62)  23.257 ms  23.282 ms  23.204 ms
 8  100ge3-2.core1.lax2.he.net (184.104.197.61)  23.396 ms  23.230 ms  23.691 ms
 9  ve959.core2.phx2.he.net (184.104.196.186)  31.957 ms  32.184 ms *
10  100ge11-1.core1.dal1.he.net (184.105.223.225)  43.229 ms  60.035 ms  43.079 ms
11  100ge7-2.core1.mci3.he.net (184.105.64.214)  63.161 ms  52.464 ms  51.799 ms
12  100ge8-1.core2.chi1.he.net (184.105.81.210)  54.575 ms  54.617 ms  55.393 ms
13  port-channel4.core3.chi1.he.net (184.104.196.218)  61.473 ms  55.425 ms  55.477 ms
14  100ge6-1.core1.tor1.he.net (184.105.80.6)  63.996 ms  63.750 ms  64.530 ms
15  t02.toroc1.on.ca.sn11.net (162.252.204.3)  81.228 ms  82.886 ms  82.302 ms
16  * * *
17  * * *
18  * * *
19  * * *
20  he.rides.across.the.nation (162.252.205.134)  99.955 ms  103.236 ms  101.589 ms
21  the.thoroughbred.of.sin (162.252.205.135)  107.168 ms  106.692 ms  106.536 ms
22  he.got.the.application (162.252.205.136)  113.152 ms  113.084 ms  112.929 ms
23  that.you.just.sent.in (162.252.205.137)  118.086 ms  117.192 ms  117.388 ms
24  it.needs.evaluation (162.252.205.138)  121.211 ms  119.357 ms  122.270 ms
25  so.let.the.games.begin (162.252.205.139)  125.559 ms  127.636 ms  127.947 ms
26  a.heinous.crime (162.252.205.140)  129.662 ms  129.854 ms  129.195 ms
27  a.show.of.force (162.252.205.141)  135.239 ms  135.807 ms  135.577 ms
28  a.murder.would.be.nice.of.course (162.252.205.142)  141.211 ms  141.149 ms  140.003 ms
29  bad.horse (162.252.205.143)  144.693 ms  145.991 ms  146.159 ms
30  bad.horse (162.252.205.144)  150.150 ms  149.814 ms  150.477 ms
31  bad.horse (162.252.205.145)  155.402 ms  156.119 ms  156.332 ms
32  he-s.bad (162.252.205.146)  161.022 ms  160.751 ms  159.827 ms
33  the.evil.league.of.evil (162.252.205.147)  164.464 ms  164.947 ms  165.165 ms
34  is.watching.so.beware (162.252.205.148)  173.116 ms  172.326 ms  172.958 ms
35  the.grade.that.you.receive (162.252.205.149)  177.454 ms  176.391 ms  176.621 ms
36  will.be.your.last.we.swear (162.252.205.150)  181.639 ms  181.433 ms  182.493 ms
37  so.make.the.bad.horse.gleeful (162.252.205.151)  184.959 ms  186.756 ms  187.289 ms
38  or.he-ll.make.you.his.mare (162.252.205.152)  191.653 ms  191.673 ms  191.829 ms
39  o_o (162.252.205.153)  197.273 ms  197.294 ms  198.032 ms
40  you-re.saddled.up (162.252.205.154)  202.039 ms  201.965 ms  200.177 ms
41  there-s.no.recourse (162.252.205.155)  207.815 ms  207.926 ms  208.132 ms
42  it-s.hi-ho.silver (162.252.205.156)  210.196 ms  209.998 ms  210.012 ms
43  signed.bad.horse (162.252.205.157)  210.607 ms  210.950 ms  210.963 ms
```

* Once we get onto the 162.252.205.x subnet (starts at .134) we have gotten to a set of routers owned and operated by one person.
    * These could also potentially be virtualized on the same host...
    * Either way, the owner of this network has manually configured the routing table on all this (virtualized?) nodes so that traffic ALWAYS goes through the network to the final destination (162.252.205.157) following the same path. 
        * And he has given these computers names to match the lyrics of the song. 