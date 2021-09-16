# IP Addressing Exercise

## Part 1: Examining Routes

Traceroute is a program that can tell you which path (in terms of IP addresses). We're going to use `traceroute` and `dig` to examine some paths from our computer to other nodes in the network.

1. Use dig to find an IPv4 address for google.com.
2. Use traceroute to discover the the path from your computer to that ip address.
3. Look carefully at the output and...
    * Identify how many hops it took to get to google.
    * Try and identify how many autonomous systems are represented.
        * Hint: use the `-a` option of traceroute to include AS numbers!
    * Your output might sometimes include *'s, what does this mean?
    * Your output might sometimes include multiple IP addresses "per hop", what does this mean?

## Part 2: Engineers Just Wanna Have Fun

* Execute the following command

```
traceroute bad.horse
```

* This might take a minute or two to complete. In the meantime get some context for this question by watching this youtube video from Dr. Horrible's Sing-Along Blog: [https://www.youtube.com/watch?v=VNhhz1yYk2U](https://www.youtube.com/watch?v=VNhhz1yYk2U)

* Look carefully at the IP addresses, then try to decide how this outcome was achieved.