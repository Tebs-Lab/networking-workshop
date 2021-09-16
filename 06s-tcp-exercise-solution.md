# TCP Exercise

In this exercise you'll be asked to explore some of TCP's behavior in more detail, including TCP's mechanisms for:

* Connection establishment,
* Flow control,
* And congestion control.

## Part 1: The "Handshake"

In the file 06x-handshake.txt you can see an example of the "three way handshake." This file was produced by a program called Wireshark, and instead of the raw binary data, it is showing you the parsed out values from the TCP header. All of the other portions of the data (i.e. headers from other layers and all payload data) have been removed so you can focus on TCP.

First, read this short article about the so called "3-way handshake": [https://www.guru99.com/tcp-3-way-handshake.html](https://www.guru99.com/tcp-3-way-handshake.html)

Then, looking at the text data, try to answer the following questions:

1. What are the port numbers being used by the client and the server?
    * client is using port 60021
    * server is using port 80

2. Based on the port numbers, can you guess what application layer protocol is going to be used?
    * HTTP data, since the server is listening on port 80.
    * Why is only one of the two port numbers a "well known" / "reserved" port number?
        * Since the server is an HTTP server, the protocol is already established simply by the server using port 80. 
        * Furthermore, the client might already have it's own HTTP server listening on port 80.
        * TCP clients typically use one of the "ephemeral ports" — i.e. ports with no established or reserved protocol. These numbers are generated mostly randomly (within the allowed range). But the TCP client must be careful not to reuse port numbers for different TCP connections.


3. Extract just the sequence and acknowledgment numbers from the three segments. What are their values, and how do they relate to each other?
    * On a per segment basis...
    * Segment 1: SEQ: 408329057 ACK: 0
    * Segment 2: SEQ: 2610001420 ACK: 408329058
    * Segment 3: SEQ: 408329058 ACK: 2610001421
    * The initial sequence numbers are chosen randomly by the client and server respectively. 
    * The initial ACK is 0, because the initial request cannot possibly acknowledge any data sent by the server as the server hasn't sent any yet.
    * In segment two the ACK is the clients initial SEQ+1
    * In segment three the ACK is the servers initial SEQ+1
    * In future requests where actual data is being transmitted, sequence numbers will increase by the size of the payload data in that particular segment.
    * ACK's always acknowledge a value 1+SEQ which can be interpreted as which byte number that host is now waiting for.

4. The client and server both advertise their "maximum segment size" (MSS), which is the largest size of a single segment that the other host is allowed to send. 
    * What is the largest segment the client is willing to receive?
        * 1460 bytes
    * What is the largest segment the server is willing to receive?
        * 1430 bytes


5. The client and server always advertise their current "receive window" in every segment.
    * What is the client's initial receive window? (in bytes)
        * 65535 bytes
    * What is the server's initial receive window?
        * 65535 bytes
    * In the 3rd segment (the ACK) the clients receive window has changed
        * What is it now? 
            * 131840
        * Is there something meaningful about the amount by which it has increased?
            * It has doubled, in accordance with the rules of Slow Start.
        * Hint: read about TCP's "slow start" mechanism: [https://www.keycdn.com/support/tcp-slow-start](https://www.keycdn.com/support/tcp-slow-start)


6. The timestamp options are being used in this handshake, there is enough information contained in them to compute the round trip time (RTT) (aka, the amount of time it took for the client to send it's request and receive a response.)
    * How long was this RTT?
        * First client timestamp: 499847865 
        * Second client timestamp: 499847900
        * 499847900 - 499847865 = 35ms (VERY FAST!)

    * Suppose the the next segment transmitted by the server had a timestamp TVal of 2523488400, what RTT would that indicate for how long it took between the servers first message (the SYN/ACK) and the this new segment?
        * 2523488400 - 2523488345 = 55ms (still fast, but slower)
        * (Fun fact, in reality, this value was 2523488380, meaning another 35ms exactly, which is kinda cool)

## Part 2: Congestion Control

In this section of the exercise, you're being asked to perform some research about TCP's congestion control mechanisms. These mechanisms are designed to ensure that TCP is using the available bandwidth in the network effectively. This sometimes means scaling up the connection size to maximize throughput on a fast connection, and it sometimes means 

1. Can you describe TCP's "slow start" mechanism?
    * How does TCP scale up the amount of data being transmitted during slow start?
    * When does TCP exit "slow start" mode, and how does it scale up or down the amount of data sent after slow start has ended?

Slow start begins transmitting data by sending 1 segment over the wire (generally at the maximum segment size, unless there is less data than that to transmit in total). It waits to recieve an ack for that data before sending anything else, even if the clients RWND is significantly larger.

If it receives an ACK for that data, it then sends 2 segments and waits for an ACK. Indeed, for every ACK'd segment, the sender increases the amount of data it is allowed to send by 1 MSS. This is exponential growth! The first segment gets ACK'd, now we can send 2 MSS at a time. When both of those are ACK'd, we will be allowed to send 4 MSS at a time. Then 8, then 16, and so on until we experience packet loss or reach the "Slow Start Threshold", which is some fixed amount of data that is allowed to be unacknowledged on the wire (this value is computed during the handshake process).

If we reach the SSThreshold value, we begin scaling up linearly. Specifically, every new acknoledged packet increases the congestion window size by 1/MSS. So if the current window is `4*MSS`, then it takes 4 consecutive ACK's to increase it to 5*MSS, whereas during slow start each of those ACKS would have increased the window by 1 MSS (and we'd have gone from 4MSS to 8MSS).


2. There are two ways that TCP detects packet loss. A "timeout" also sometimes called a "major packet loss" event, and a series of 3 duplicate acknowledgments, sometimes called a "minor packet loss" event.
    * Why is a timeout considered to be a stronger sign of more significant problems in the network compared to a series of 3 duplicate ACKs?
    * Can you describe a series of events that would result in a timeout?
    * Can you describe a series of events that would result in 3 duplicate acks?

A timeout occurs when one of the hosts has not received ANY segments from the other host for some period of time (the timeout value is determined during the handshake, typically based on the computed RTT). A duplicate ACK occurs when one of the hosts sends 3 segments in a row with the same ACK value.

A timeout is considered more significant because if ACK's are being received it suggests that the underlying connection is still working (even though one packet seems to have been lost). A timeout indicates that NO data is flowing on the network, so is considered more significant. 

A series of events that could lead to timeout are: one of the hosts goes offline, or one of the nodes in the network transmitting the data goes offline and there is no longer a path between the two hosts. (this is a non-exhaustive list).

A series of events that could lead to 3 duplicate ACKs is: one router along the path drops a segment because its buffer is full, but that node immediately recovers and therefore the rest of the data is flowing. The client always sends the ACK value for the EARLIEST MISSING SEGMENT, which would be the value for the segment that was dropped. The client is still receiving data from the server, but until the server retransmits the dropped packet (indicated by the ACK number) the client will keep sending the duplicate ACK value.