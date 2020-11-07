# Networking

The goal is to send a message, securely, over a network. There's a lot involved in that. 

### Network Adversaries

Different types of adversaries for a connection, strength dependent on what they can do with the message sent:

- **Off-path Adversaries**: cannot read or modify
- **On-path Adversaries**: can read, not modify
- **In-path Adversaries (MITM)**: read, modify, and block messages.

All adversaries can also send their own messages, including **spoofing** them. Sometimes all they need to do is set the source IP/port to something else. 

## Lower Layers: ARP and DHCP

A **Local Area Network** is a bunch of connected computers in a small area (connected via Link Layer)

**Ethernet** assigns a 6-byte MAC address to each computer on the LAN. MAC addresses = 6 pairs of hex numbers, e.g. ca:fe:f0:0d:be:ef. There **broadcast address** ff:ff:ff:ff:ff:ff broadcasts a message to all LAN computers whenever something is sent to it. 

Ethernet was originally broadcast-only: each *node* could hear all other nodes, either by being on a common wire or through a **network hub**, a repeater that broadcast everything it received to all connected output nodes. A node was morally obligated to ignore everything not directly sent to it or broadcast, but because humans are inherently evil, most Ethernet devices can enter **promiscuous mode**, where it can sniff all data packets. 

Such archaic versions of Ethernet (like any network using a hub), let's say an adversary existed as a node in the local network. He could sniff *all* traffic and can also spoof MAC addresses and generate traffic himself. 

### ARP

Our network header gives us our (global) destination IP address. At the link layer, however, everything we get our destination's *local* MAC address. **Address Resolution Protocol** (ARP) maps IP to MAC addresses. 

Let's say Alice wants to send a message to Bob at IP address 1.1.1.1. The ARP protocol would find Bob's MAC address in 3 steps:

1. Alice broadcasts a query to the LAN, asking for MAC address of 1.1.1.1
2. Bob responds by sending his IP and MAC address to Alice.
3. Alice gets message, then maps Bob's IP address to MAC address and *caches* it.

If Bob is outside the LAN, then the *gateway* (instead of Bob) responds with Bob's IP and MAC address (step 2)

Note something interesting: **any received ARP replies are always cached**, even if no broadcast request (step 1) was ever made.

### ARP Spoofing

Building off that note, there isn't actually any authenticity verification for replies in the ARP protocol! Thus it is easy to **spoof** a reply to Alice (before Bob replies) and get Alice to map Bob's IP to a malicious MAC address instead. So now Mallory receives all of Alice's messages she thinks she's sending to Bob. 

ARP spoofing is an example of a **race condition**: Mallory's response must be faster than Bob's (legit) response to fool Alice. On-path attackers utilize this a lot, as they can't actually edit/block the legit response message, and thus can only try to get their fake response in first. 

### Switches: A Defense

A tool like arpwatch tracks IP address to MAC address mappings and tries to detect any suspicious maps. 

However, **switches** are becoming a more and more common solution to ARP spoofing. Modern Ethernet networks have switched to switches (nice) over hubs. Switches have a **MAC cache**, which, like arpwatch, tracks IP address -> MAC address mappings. 

