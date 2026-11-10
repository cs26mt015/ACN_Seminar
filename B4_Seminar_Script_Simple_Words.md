# B4 Seminar --- Simple Presentation Script

**Paper:** *B4: Experience with a Globally-Deployed Software Defined
WAN* --- Sushant Jain et al., Google, ACM SIGCOMM 2013\
**Presenters:** Avishkar (CS26MT015), Tarun (CS26MT012)\
**Timing:** 15-minute presentation + 5-minute Q&A

## Presenter split

-   Avishkar: Slides 1--9 (about 7 min 15 sec)
-   Tarun: Slides 10--17 (about 7 min 05 sec)

------------------------------------------------------------------------

## Slide 1 --- Title (20 sec) --- Avishkar

"Good morning everyone. Today, we are presenting a research paper called
*B4: Experience with a Globally-Deployed Software Defined WAN*. It was
written by Sushant Jain and his team at Google and published at ACM
SIGCOMM in 2013.

This paper explains how Google built a private network to connect its
data centers and how it used the network bandwidth more efficiently."

## Slide 2 --- The Problem with Traditional WANs (50 sec) --- Avishkar

"Traditional wide area networks are built with extra capacity so they
can keep working if a link or router fails.

The paper says that traditional WAN links often used only around 30 to
40 percent of their capacity on average. This means much of the
available capacity stayed unused during normal operation.

The paper also says that networks often had two to three times more
bandwidth than average traffic needed. They also needed expensive,
high-end routers.

This extra capacity helps the network stay reliable, but it also
increases cost. Google wanted to use its network capacity better."

## Slide 3 --- Why Google's WAN Was Different (45 sec) --- Avishkar

"Google's network had three things that made a new approach possible.

First, Google had a large amount of traffic moving between its data
centers. Examples include copying data and sharing search index updates.

Second, many applications were flexible. If more bandwidth was
available, they could use it to send data faster.

Third, Google controlled its network, servers and applications.

Because of these three things, Google could manage traffic from one
central system and use the available bandwidth better."

## Slide 4 --- Introducing B4 (45 sec) --- Avishkar

"To solve this problem, Google built a private WAN called B4. It
connects Google's data centers in different locations.

B4 uses Software Defined Networking, or SDN, and a system called Traffic
Engineering. A central system decides how applications should use the
available network paths.

The paper reports that B4 links were used close to their full capacity
for long periods.

The main idea is to share the available bandwidth between applications
in a smart way."

## Slide 5 --- Why Software Defined Networking? (45 sec) --- Avishkar

"Software Defined Networking separates the part that makes network
decisions from the devices that send packets.

In a traditional network, routers usually make their own decisions using
the information they have. This works for finding paths, but it is
harder to manage traffic across the whole WAN and share bandwidth
between applications.

With SDN, a central controller can get a wider view of the network and
make decisions for the whole network.

This was useful for Google because it controlled its network, and its
applications could use more bandwidth when it was available."

## Slide 6 --- B4 Architecture: Three Layers (60 sec) --- Avishkar

"B4 has three main layers.

The first is the global layer. It has the Traffic Engineering server,
which works out how traffic should be shared. The SDN Gateway connects
this central system to the OpenFlow system at each site. The service is
central in its design, but it runs on multiple servers. A leader is
chosen to do the main work.

The second is the site control layer. It has Network Control Servers,
OpenFlow Controllers and Network Control Applications. They manage
network control at each site.

The third is the switch layer. The OpenFlow Agent receives instructions
and passes them to the switch hardware.

So, the global layer makes the plan, the site controllers pass on the
instructions, and the switches send the packets."

## Slide 7 --- Routing and Traffic Engineering (50 sec) --- Avishkar

"Routing and Traffic Engineering are not the same.

Routing finds which destinations can be reached and which paths are
available. B4 still uses normal routing protocols such as BGP and IS-IS.

Traffic Engineering decides how traffic should be shared across those
paths. It considers how much bandwidth applications need.

So B4 does not remove traditional routing. It adds another system on top
of it. Routing finds the paths, and B4 decides how to use them better."

## Slide 8 --- Flow Groups, Tunnels and Tunnel Groups (60 sec) --- Avishkar

"B4 uses three important ideas.

A Flow Group, or FG, is a way to group traffic. It is identified by the
source site, destination site and Quality of Service, or QoS.

A tunnel is a path through the WAN. For example, traffic may go from
site A, through site B, and then to site C. B4 makes these tunnels using
a method called IP-in-IP encapsulation.

A Tunnel Group, or TG, connects one Flow Group to one or more tunnels.
Each tunnel gets a weight. The weight tells us what part of the Flow
Group's traffic should use that tunnel.

In simple words: Flow Group tells us which traffic we are handling, a
tunnel tells us the path, and a Tunnel Group tells us how to divide the
traffic across paths."

## Slide 9 --- How B4 Allocates Bandwidth (60 sec) --- Avishkar

"Suppose one Flow Group uses three tunnels. B4 may send 50 percent of
its traffic through the first tunnel, 40 percent through the second, and
10 percent through the third.

B4 decides these shares by looking at application needs, priorities and
the available network capacity.

The aim is to share bandwidth fairly and use the network links well. The
switches then send packets through the tunnels using the shares decided
by B4.

Now we will see how B4 works out these shares."

**Handoff:** "I will now hand over to Tarun. He will explain how B4
calculates and sends these traffic instructions, and what the team
learned from using B4 in the real network."

## Slide 10 --- B4 Traffic Engineering Algorithm (55 sec) --- Tarun

"The algorithm takes information about the network, link capacity and
traffic needs as input.

It creates Tunnel Groups by choosing paths and giving each path a share
of the traffic. Then it changes these shares into values that the switch
hardware can use. This step is called quantization.

Finding the exact best answer can take too much time for a network of
this size. So B4 uses a faster method that gives a result close to the
best answer.

In the paper, the average time for this calculation was about 0.3
seconds. The longest reported time was about 0.8 seconds."

## Slide 11 --- How Tunnels Work in the Data Plane (55 sec) --- Tarun

"Now let us see how packets travel through a tunnel.

At the source, the first switch identifies the Flow Group and chooses a
tunnel. It adds a new outer IP header around the original packet. The
outer destination tells the network which tunnel to use.

The switches in the middle use this outer header to send the packet
along the tunnel.

At the destination, the last switch removes the outer header. It then
sends the original packet towards its final destination using normal
routing.

This lets B4 choose the path for traffic while keeping the original
packet inside."

## Slide 12 --- Integrating B4 with Traditional Routing (50 sec) --- Tarun

"B4 works together with the existing routing system.

BGP and IS-IS provide information about which destinations can be
reached. This information goes to the routing system. In B4's
implementation, Quagga is used as part of the routing software, and RAP
connects the traditional routing system with OpenFlow control.

If a packet matches a B4 Traffic Engineering rule, the switch uses the
multipath and tunnel tables to send it through the selected tunnels.

If the packet does not match a Traffic Engineering rule, it uses normal
routing based on the longest matching destination prefix.

This is how B4 adds traffic control while still keeping traditional
routing."

## Slide 13 --- B4 in Production (50 sec) --- Tarun

"B4 was used in Google's real network. It was not just a test system.

During the period measured in the paper, B4 had 16 sites, around 2,700
Flow Groups and 350 tunnels in use. It used five Traffic Engineering
servers in different locations.

It also had backup servers that could take over the main job in less
than ten seconds.

The Traffic Engineering calculation was very fast. But sending
instructions to switches and making changes in the network could take
more time. This becomes important when we look at what happens during
failures."

## Slide 14 --- Performance Results (65 sec) --- Tarun

"The paper compares B4 with normal shortest-path routing.

It reports that B4 improved average throughput by about 14 percent.
Adding more paths helped at first, but the improvement became smaller
after around four paths.

In the real network, Google used four paths and a one-quarter
traffic-splitting step. This was a balance between better performance
and the limited resources inside switches.

The paper also gives a separate example involving a high-priority data
copy over one week. In that example, network use improved by about five
percent, and the amount of network capacity needed could be reduced a
lot.

Remember that these are two different results: 14 percent is the average
throughput improvement, while about five percent is from the separate
weekly example."

## Slide 15 --- Failure Handling (45 sec) --- Tarun

"The paper also checks how much packet loss happened when different
parts of B4 failed.

When one link failed, packet loss lasted about four milliseconds. When
an encapsulation switch failed, it lasted about ten milliseconds.

But when a transit switch near an encapsulation switch failed, packet
loss lasted much longer: about 3.3 seconds. The nearby encapsulation
switch had to update many tunnel entries, and these updates took time.

In the tests described in the paper, failure of an OpenFlow Controller,
failure of the Traffic Engineering server, or turning Traffic
Engineering off and on caused zero measured packet loss.

The main lesson is that it is not enough for B4 to calculate a new plan
quickly. The network must also be able to update the switches quickly."

## Slide 16 --- A Real Production Outage (75 sec) --- Tarun

"The paper describes a real network problem that happened during planned
maintenance at Google's largest site.

A new switch was given the same ID as a switch that was already there.
This caused problems in the routing system and made the network send and
process too many control messages.

The messages built up in a queue between the OpenFlow Controller and the
OpenFlow Agent. This queue grew to more than 400 megabytes.

Because the messages were delayed, IS-IS Hello messages were not handled
in time. Some switches then thought their links were down. Connections
using BGP to other sites were also lost.

At first, the team tried restarting the OpenFlow Controller servers. But
the heavy load exposed a hidden software problem, so the controllers did
not recover properly.

To fix the problem, the team moved traffic away from the site, turned
off Traffic Engineering and restarted the controllers from the
beginning.

This problem showed that the system controlling the network must also be
fast and reliable. The paper lists lessons such as handling messages in
the background using multiple threads, checking system speed, allowing a
safe fallback option and testing the system under very heavy load."

## Slide 17 --- Conclusion (30 sec) --- Tarun

"To conclude, B4 showed that central control can help a private WAN use
its bandwidth much better.

It combined SDN and OpenFlow with traditional routing, application-based
bandwidth sharing and tunnels.

But the real network experience also showed that the control system is
very important. A fast calculation is not enough if sending instructions
to switches or fixing failures takes too long.

The main message is that B4 helped Google use its network capacity
better, but the system controlling the network must also be reliable.

Thank you. We are happy to answer your questions."

------------------------------------------------------------------------

# Likely Q&A --- Simple Answers

**1. What is B4?**\
B4 is Google's private network that connects its data centers. It uses a
central system to share network bandwidth between applications.

**2. Why did Google build B4?**\
Traditional WANs keep a lot of extra capacity for safety. Google wanted
to use more of its available network capacity for traffic between its
data centers.

**3. What is SDN?**\
SDN is a way of managing a network where the part that makes decisions
is separated from the devices that send packets. A central controller
can help manage the network.

**4. Does B4 replace BGP and IS-IS?**\
No. B4 still uses BGP and IS-IS for normal routing. It adds its own
Traffic Engineering system on top.

**5. What is the difference between routing and Traffic Engineering?**\
Routing finds the available paths. Traffic Engineering decides how much
traffic should use each path.

**6. What is a Flow Group?**\
A Flow Group is a group of traffic with the same source site,
destination site and Quality of Service.

**7. What is a Tunnel Group?**\
A Tunnel Group connects a Flow Group to one or more tunnels. It also
decides what share of traffic should use each tunnel.

**8. Why does B4 use tunnels?**\
Tunnels let B4 send traffic along paths that it has selected, instead of
always using the normal route chosen for the destination.

**9. What is IP-in-IP encapsulation?**\
It means adding a new IP header around the original packet. The new
outer header helps the network send the packet through the selected
tunnel. At the end, the outer header is removed.

**10. What does quantization mean?**\
It means changing traffic shares into fixed steps or values that the
switch hardware can use.

**11. What is the main performance result?**\
The paper reports about 14 percent better average throughput compared
with shortest-path routing in its evaluation.

**12. Why did Google use four paths instead of many more?**\
The benefit became smaller after around four paths. More paths also use
more space in the switches. Four paths gave a useful balance.

**13. Why did one failure cause 3.3 seconds of packet loss?**\
A failed transit switch made a nearby encapsulation switch update many
tunnel entries. These updates took time.

**14. What caused the production outage?**\
During maintenance, a new switch was given an ID that was already being
used. This caused routing problems, too many control messages, a large
queue and delayed network messages.

**15. What is the biggest lesson from the outage?**\
The system that controls the network must work reliably and quickly. It
is important to test it under heavy load and make sure it can recover
from problems.

**16. Can B4 be used on the public Internet?**\
Not in exactly the same way. B4 was built for a network that Google
could control. The public Internet is made up of many networks owned and
managed by different organizations.

**17. Is 100 percent link use always good?**\
Not always. The paper reports that some B4 links were used close to
their full capacity. A network still needs to handle failures, changing
traffic and important applications.

**18. What is the difference between the 14 percent and 5 percent
results?**\
The 14 percent result is the average improvement in throughput in the
evaluation. The roughly 5 percent result comes from a separate example
over one week and refers to better network use.

------------------------------------------------------------------------

## Important points to remember

-   Do not say B4 removed BGP or IS-IS. It continued to use them.
-   The Traffic Engineering system was central in design, but it ran on
    multiple servers. It was not just one physical server.
-   Do not say every link was always exactly 100 percent full.
-   Keep the 14 percent throughput result separate from the roughly 5
    percent weekly example.
-   The failure table shows how long packet loss lasted. It does not
    simply show how long it took to detect a failure.
-   Remember the outage order: duplicate switch ID → routing problems →
    queue became too large → IS-IS Hello messages were delayed →
    connections were lost → recovery steps.
