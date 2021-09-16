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
2. Based on the port numbers, can you guess what application layer protocol is going to be used?
    * Why is only one of the two port numbers a "well known" / "reserved" port number?
    * Hint: search for "ephemeral ports" 
3. Extract just the sequence and acknowledgment numbers from the three segments. What are they, and how do they relate to each other?
4. The client and server both advertise their "maximum segment size" (MSS), which is the largest size of a single segment that the other host is allowed to send. 
    * What is the largest segment the client is willing to receive?
    * What is the largest segment the server is willing to receive?
5. The client and server always advertise their current "receive window" in every segment.
    * What is the client's initial receive window? (in bytes)
    * What is the server's initial receive window?
    * In the 3rd segment (the ACK) the clients receive window has changed
        * What is it now? 
        * Is there something meaningful about the amount by which it has increased?
        * Hint: read about TCP's "slow start" mechanism: [https://www.keycdn.com/support/tcp-slow-start](https://www.keycdn.com/support/tcp-slow-start)
6. The timestamp options are being used in this handshake, there is enough information contained in them to compute the round trip time (RTT) (aka, the amount of time it took for the client to send it's request and receive a response.)
    * How long was this RTT?
    * Suppose the the next segment transmitted by the server had a timestamp TVal of 2523488400, what RTT would that indicate for how long it took between the servers first message (the SYN/ACK) and the this new segment?

## Part 2: Congestion Control

In this section of the exercise, you're being asked to perform some research about TCP's congestion control mechanisms. These mechanisms are designed to ensure that TCP is using the available bandwidth in the network effectively. This sometimes means scaling up the connection size to maximize throughput on a fast connection, and it sometimes means 

1. Can you describe TCP's "slow start" mechanism?
    * How does TCP scale up the amount of data being transmitted during slow start?
    * When does TCP exit "slow start" mode, and how does it scale up or down the amount of data sent after slow start has ended?

2. There are two ways that TCP detects packet loss. A "timeout" also sometimes called a "major packet loss" event, and a series of 3 duplicate acknowledgments, sometimes called a "minor packet loss" event.
    * Why is a timeout considered to be a stronger sign of more significant problems in the network compared to a series of 3 duplicate ACKs?
    * Can you describe a series of events that would result in a timeout?
    * Can you describe a series of events that would result in 3 duplicate acks?