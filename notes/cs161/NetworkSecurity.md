# Network Security

Networks are beautiful. You've got a network of people you know closely and talk to often. You're very likely to be familiar with computer networks as well! A network is what allows us to send snaps, share documents, and send huge amounts of information nearly instantaneously to others. 

Alas, with all good things come evil: computer networks also open up our machines to attackers. We need to implement some form of security.

*Network security* is quite a broad term and concerns a lot of different levels. Historically, only system administrators cared about this sort of thing. Now, however, networks are becoming pretty much ubiquitous, so developers should know about this stuff too. For example, pretty much all modern applications contain network-facing parts and therefore **must** be tested before attackers can get a chance to send malicious code. 

In this note, we'll explore some fundamental concepts of networks and network security.

### LANs and WANs

At the smallest, physical (electrical) level, we see two means of connection: **Local Area Networks (LANs)** and **Wide Area Networks (WANs)**. In these connections, we'll call individual computers *hosts*.

Local Area Networks are fairly self-explanatory: they are local to an area limited in size. Typically, LANs provide high-speed connections across the local area spanned; however, their high performance usually comes with being more expensive. If you're any type of online gamer you'll certainly be familiar with a type of LAN: Ethernet! Traditionally, Ethernet-based LANs operate as **broadcast protocols**, meaning whenever a single machine on the LAN wants to send some message (a *data packet* in technical terms), every machine on the LAN can see it. You can see how this might cause privacy concerns; consequently, more and more businesses are now switching to *switched LANs*, which utilizes a switch to effectively give each host its own LAN. To connect LANs to other networks (including other LANs), the connections must pass through a *gateway* machine. 

Similarly, Wide Area Networks span a wider area. Conversely to LANs, WANs trade off performance for cost reduction. We're also pretty familiar with WANs: these include things like phone lines and satellite connections. Unfortunately, the larger-area thing means far more complex and less-organized network graphs, which means that even a single fuck-up can lead to major disruptions. 

### Finding Hosts on a Network 

In order to send stuff over a network, we have to first locate the machines to send stuff to. We have to figure out how to name these hosts: we fittingly call these names **hostnames**, which are simply human-understandable names. For another example, think about website URLs, which are in themselves a (massive) network: the best URL on the planet, <code>www.cs61a.org</code>, is an example of a hostname.

Next up is the **IP Address**, which assigns each host a machine-understandable name. IPv4 uses 32-bit addresses split up into bytes separated by dots. However, this only offers 2^32 possible addresses, and very soon there'll be more computers in the world than that. Thus IPv6 uses 128-bit addresses and offers plenty more. Do IP addresses map one-to-one with hosts? Unfortunately, not in the real world. *Multihomed* machines can hog multiple IP addresses. The **Dynamic Host Configuration Protocol** (DHCP) assigns each host a different IP address every time it goes online, and can re-use IP addresses for different machines. Your household might even utilize **Network Address Translation** (NAT) to allow multiple LAN hosts (your computer, your kid's iPhone) to have the same IP address *from the perspective of those outside*.

We, as humans, specify hostnames that we want to send stuff to. Then computers find the IP address that map to this hostname. For our URL example, if we want to visit <code>www.cs61a.org</code> (and we *certainly* want to), the **Domain Name System** (DNS) serves as the means to find our IP address. In DNS we have a hierarchy of **nameservers** that recursively ask for subdomains starting from the top-level domain (TLD) until we find our full hostname (and mapped IP address). That's a mouthful; let's walk through the process. First we ask the top-level nameserver, which tells us the "org" nameserver. The "org" nameserver tells our machine where the "cs61a.org" nameserver is. cs61a.org is a **subdomain** of ".org". This is a gross simplification of DNS, however: there are other properties like **caching**, in which if we already visited a site recently the DNS would simply fetch that site's IP address from the cache without even touching a nameserver. 

So now we found our IP address. Now our host needs to find a **route** to the host at that address. Similar to fetching IP addresses in DNS through nameservers, *routing* utilizes a hierarchical process to find a valid route, and also utilizes several **routing protocols** (rules) along the way. Within enterprise LANs, simpler protocols like *Open Shortest Path First* (OSPF) provide a framework for constructing routes. Across the internet, however, the *Border Gateway Protocol* (BGP) is used instead.

