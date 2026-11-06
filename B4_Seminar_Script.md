# B4 Seminar --- Presentation Script

**Paper:** *B4: Experience with a Globally-Deployed Software Defined
WAN* --- Sushant Jain et al., Google, ACM SIGCOMM 2013\
**Presenters:** Tarun (CS26MT012) and Avishkar (CS26MT015)\
**Duration:** 15 minutes presentation + 5 minutes Q&A

## Presenter split

-   **Tarun:** Slides 1--9 (about 7 min 15 sec)
-   **Avishkar:** Slides 10--17 (about 7 min 05 sec)
-   Target around 14 min 20 sec to leave a small transition buffer.

------------------------------------------------------------------------

## Slide 1 --- Title (20 sec) --- Tarun

"Good morning everyone. Today, we are presenting the paper *B4:
Experience with a Globally-Deployed Software Defined WAN*, published by
Sushant Jain and colleagues from Google at ACM SIGCOMM 2013. This paper
describes how Google designed and deployed a software-defined wide area
network to use bandwidth more efficiently between its data centers."

## Slide 2 --- The Problem with Traditional WANs (50 sec) --- Tarun

"Traditional WANs are generally provisioned with reliability in mind.
Operators reserve spare capacity so that the network can continue
operating when a link or router fails. The paper explains that
traditional WAN links were often provisioned for around 30 to 40 percent
average utilization. It also describes two to three times bandwidth
over-provisioning and expensive, high-end routing equipment. This
approach provides a reliability margin, but it means substantial
capacity may remain unused during normal operation. Google wanted to use
this capacity more efficiently."

## Slide 3 --- Why Google's WAN Was Different (45 sec) --- Tarun

"Google's environment had three characteristics that made a different
approach possible. First, Google had large volumes of traffic between
geographically distributed data centers, such as data replication and
index updates. Second, many applications were elastic: they could use
more bandwidth when it became available. Third, Google controlled its
network infrastructure, edge servers and applications. Together, these
characteristics made centralized traffic management practical."

## Slide 4 --- Introducing B4 (45 sec) --- Tarun

"To address this problem, Google built B4, a private WAN connecting its
geographically distributed data centers. B4 uses Software Defined
Networking and centralized Traffic Engineering. Instead of relying only
on local routing decisions, a central system computes how application
traffic should use the available network paths. The paper reports that
B4 links operated close to full utilization over long periods. The idea
is to allocate available bandwidth intelligently among applications."

## Slide 5 --- Why SDN? (45 sec) --- Tarun

"Software Defined Networking separates network control logic from the
devices that forward packets. In a traditional distributed approach,
routers make decisions based largely on local routing information. This
works for reachability, but it is harder to coordinate application-level
bandwidth allocation across the WAN. With SDN, a logically centralized
controller can obtain a broader view and compute coordinated decisions.
This suited Google because it controlled the network and had
applications that could adapt to available bandwidth."

## Slide 6 --- B4 Architecture (60 sec) --- Tarun

"B4's architecture has three layers. At the global layer, the Traffic
Engineering server computes traffic allocations. The SDN Gateway
provides an abstraction between the global system and site-level
OpenFlow infrastructure. The service is logically centralized but
replicated across sites, with leader election selecting the active
primary. At each site, Network Control Servers, OpenFlow Controllers and
Network Control Applications coordinate local control. At the switch
layer, the OpenFlow Agent receives commands and translates them into
operations that program the switch hardware. The global layer decides,
site controllers coordinate, and switches forward packets."

## Slide 7 --- Routing and Traffic Engineering (50 sec) --- Tarun

"Routing determines which destinations are reachable and provides paths
through the network. B4 retains conventional routing protocols such as
BGP and IS-IS. Traffic Engineering decides how traffic should be
distributed over those available paths, considering bandwidth and
application requirements. B4 does not simply replace traditional
routing. It builds an optimization layer on top of it: routing provides
paths, and B4 decides how to use them efficiently."

## Slide 8 --- Flow Groups, Tunnels and Tunnel Groups (60 sec) --- Tarun

"A Flow Group, or FG, identifies a category of traffic using its source
site, destination site and Quality of Service. A tunnel represents a
path through the WAN at the site level. For example, traffic may travel
from site A through site B to site C. B4 implements these tunnels using
IP-in-IP encapsulation. A Tunnel Group, or TG, maps a Flow Group to a
set of tunnels. Each tunnel has a weight that determines what fraction
of the Flow Group's traffic should use it. In short: the Flow Group
identifies the traffic, a tunnel identifies a path, and the Tunnel Group
defines how traffic is split across paths."

## Slide 9 --- How B4 Allocates Bandwidth (60 sec) --- Tarun

"Suppose one Flow Group is assigned three tunnels. B4 might allocate 50
percent of its traffic to the first tunnel, 40 percent to the second and
10 percent to the third. Allocation depends on application demand,
priorities and available capacity. B4 aims to share bandwidth fairly
while using links efficiently. The forwarding system then distributes
packets across tunnels according to the computed weights. This brings us
to the algorithm that calculates those allocations."

**Handoff:** "I'll now hand over to Avishkar, who will explain how B4
computes and implements these allocations, and what happened in
production."

## Slide 10 --- B4 Traffic Engineering Algorithm (55 sec) --- Avishkar

"The algorithm takes network topology, link capacities and Flow Group
demands as input. It generates Tunnel Groups by selecting paths and
assigning weights. The ratios are then quantized, meaning converted into
values the forwarding hardware can implement efficiently. Solving the
exact optimization problem can be expensive at this scale, so B4 uses an
algorithm designed to achieve similar bandwidth efficiency with lower
computation time. The reported average computation time was around 0.3
seconds, with a maximum around 0.8 seconds."

## Slide 11 --- How Tunnels Work (55 sec) --- Avishkar

"At the source, or encapsulating switch, B4 identifies the Flow Group,
selects a tunnel and adds an outer IP header. The outer destination
identifies the tunnel. Transit switches forward using this outer header.
At the destination, the outer header is removed and the original packet
is forwarded using normal routing. This lets B4 steer traffic through
selected paths while preserving the original packet."

## Slide 12 --- Integrating B4 with Traditional Routing (50 sec) --- Avishkar

"B4 integrates its traffic engineering rules with the existing routing
system. BGP and IS-IS provide reachability information to the routing
stack. In the implementation, Quagga is used as part of the routing
software, and RAP bridges traditional routing with OpenFlow control.
When a packet matches a Traffic Engineering rule, the switch uses
multipath and tunnel tables to distribute it across configured tunnels.
If there is no Traffic Engineering match, the packet can use normal
longest-prefix-match routing. B4 combines centralized optimization with
conventional routing rather than replacing it."

## Slide 13 --- B4 in Production (50 sec) --- Avishkar

"B4 was deployed as a real production network. The paper's measurement
period describes 16 sites, around 2,700 Flow Groups and 350 tunnels in
use. The system used five geographically distributed Traffic Engineering
servers. Hot standbys could take over mastership in less than ten
seconds. The Traffic Engineering computation itself was fast. However,
programming switches and interacting with the control infrastructure
could contribute significant latency. This distinction matters when we
look at failures."

## Slide 14 --- Performance Results (65 sec) --- Avishkar

"The paper evaluates B4 against shortest-path routing and reports an
average throughput improvement of about 14 percent. The benefit of
adding more paths increases at first, but gains begin to flatten at
around four paths. The production configuration used four paths and a
one-quarter traffic-splitting quantum, balancing performance and switch
resource constraints. The paper also describes a separate weekly
high-priority data-copy example that improved utilization by about five
percent and could substantially reduce required deployed capacity. These
are distinct results: 14 percent is average throughput improvement;
about five percent is the separate weekly utilization example."

## Slide 15 --- Failure Handling (45 sec) --- Avishkar

"The paper measures packet loss after different component failures. A
single link failure caused around four milliseconds of packet loss, and
an encapsulation switch failure around ten milliseconds. A transit
switch failure near an encapsulation switch caused a longer interruption
of about 3.3 seconds. The neighboring encapsulation switch had to update
multiple tunnel-related forwarding entries, and those operations were
relatively slow. In the reported experiments, OpenFlow Controller
failure, Traffic Engineering server failure and disabling or enabling
Traffic Engineering caused zero measured packet loss. Recovery depends
not only on the algorithm, but also on how quickly the control system
updates switch hardware."

## Slide 16 --- A Real Production Outage (75 sec) --- Avishkar

"The paper describes a real outage during planned maintenance at the
largest site. A newly configured switch accidentally received the same
identifier as an existing switch. This caused routing instability and
excessive link-state protocol processing. The resulting control traffic
overwhelmed a constrained communication path between the OpenFlow
Controller and OpenFlow Agent. The queue grew beyond 400 megabytes. As
routing messages were delayed, IS-IS Hello messages were not processed
in time. Switches declared interfaces down, and BGP adjacencies to
remote sites were also lost. Operators initially tried restarting the
OpenFlow Controller servers, but high load exposed a latent bug that
prevented recovery. Recovery involved draining the site, disabling
Traffic Engineering and restarting the controllers from scratch. The
outage showed that the control path is critical. The authors discuss
asynchronous and multithreaded I/O, performance profiling, fail-open
behavior and stronger stress testing."

## Slide 17 --- Conclusion (30 sec) --- Avishkar

"To conclude, B4 demonstrated that centralized Traffic Engineering can
use the capacity of a controlled WAN much more efficiently. It combined
SDN and OpenFlow with traditional routing, application-aware bandwidth
allocation and tunnel-based forwarding. The production experience also
showed that the control system itself becomes critical to network
reliability. Fast optimization is not enough if switch programming and
failure recovery are slow. Thank you. We are happy to take questions."

------------------------------------------------------------------------

# Likely Q&A --- Short Answers

**1. What is B4?**\
Google's private software-defined WAN connecting its data centers, using
centralized Traffic Engineering to allocate bandwidth.

**2. Why did Google build B4?**\
Traditional WANs reserve substantial spare capacity for reliability.
Google wanted to use its controlled private WAN capacity more
efficiently for elastic data-center traffic.

**3. What is SDN?**\
An architecture that separates control logic from packet forwarding and
enables logically centralized network decisions.

**4. Does B4 replace BGP and IS-IS?**\
No. B4 retains traditional routing protocols for reachability and adds
centralized Traffic Engineering.

**5. Routing vs Traffic Engineering?**\
Routing determines reachable paths. Traffic Engineering decides how
traffic is distributed over those paths to meet bandwidth and
application objectives.

**6. What is a Flow Group?**\
Traffic identified by source site, destination site and Quality of
Service.

**7. What is a Tunnel Group?**\
A mapping from a Flow Group to one or more tunnels, with weights
determining the traffic fraction sent through each.

**8. Why use tunnels?**\
They let B4 steer traffic along selected site-level paths independently
of ordinary destination-based forwarding.

**9. What is IP-in-IP encapsulation?**\
An outer IP header is added around the original packet. The outer header
steers the packet through the tunnel; the inner packet is recovered at
the destination.

**10. What is quantization?**\
Converting ideal traffic-splitting ratios into discrete ratios that
switch hardware can implement.

**11. Main performance result?**\
About 14 percent average throughput improvement over shortest-path
routing in the paper's evaluation.

**12. Why not use many more than four paths?**\
Benefits flattened around four paths, while additional paths and finer
splits consume forwarding-table resources. The deployed configuration
was a practical trade-off.

**13. Why was the 3.3-second failure worse?**\
A transit switch failure near an encapsulation switch required multiple
tunnel-related forwarding-table updates, which were slow in the measured
system.

**14. What caused the production outage?**\
A duplicate switch identifier during maintenance caused routing
instability, excessive control traffic, queue buildup and delayed
routing messages.

**15. Biggest lesson from the outage?**\
The control plane and switch-programming path must be engineered and
stress-tested carefully. A controller is useful only if communication
with forwarding devices remains reliable.

**16. Can B4 be used on the public Internet?**\
Not directly in the same form. B4 assumes broad control over the
network, applications and endpoints; the public Internet is operated by
many independent organizations.

**17. Is 100% utilization always desirable?**\
No. The paper reports heavily utilized links, not that every link was
always exactly full. Operators still need to account for failures,
priorities and traffic changes.

**18. Difference between 14% and 5% results?**\
The approximately 14 percent figure is average throughput improvement in
the evaluation. The approximately 5 percent figure comes from a separate
weekly high-priority copy example and refers to utilization improvement.

------------------------------------------------------------------------

## Final reminders

-   Do not say B4 replaced BGP or IS-IS; it retained traditional
    routing.
-   Do not describe the TE service as one physical server; it was
    logically centralized and replicated.
-   Do not say every link was continuously at exactly 100% utilization.
-   Keep the 14% throughput result separate from the \~5% weekly
    utilization example.
-   The failure table reports packet-loss duration, not generic
    failure-detection time.
-   Explain the outage chain clearly: duplicate switch ID → routing
    instability → control queue buildup → delayed IS-IS Hellos →
    adjacency loss → recovery actions.
