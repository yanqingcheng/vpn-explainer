# 2026-10-08: first session

Socratic session, about 13:59 to 14:42 BST. High-level pass over routing, tunnelling and what a VPN is; the chapter order in the [learning path](index.md) came out of it.

## Already knew

- A solid layer-3 picture: packets hop router to router, each router deciding the next hop from its routing table.
- Default routes carry traffic upward from the edge until it reaches the core, where routers know where everything is. (Bori misread this at first as "default routes all the way"; Qing corrected it. Still to cover: the core learns its full table through BGP.)
- Longest-prefix match: when several routes match a destination, the most specific one wins. Answered without prompting.

## Recalled with prompting

- NAT at the edge (private home addresses rewritten to one public address).
- ARP, and that the layer-2 header is rewritten at every hop while the IP header carries on.
- MPLS: labels sitting at "layer 2.5", between the layer-2 and IP headers.
- Tunnelling as the general idea of wrapping one packet inside another.

## Corrections

- **A VPN is encapsulation, not NAT.** The inner packet is wrapped in a new outer header and unwrapped at the far end; nothing is rewritten in place.
- **"Private" in MPLS VPNs meant isolation**, not encryption. Each customer's traffic is kept apart by labels and VRFs; it is not encrypted.

## Settled

- [Longest-prefix match](knowledge/longest-prefix-match.md).

## Open questions

- Why is Tailscale a mesh rather than hub-and-spoke?
- Why do VPNs prefer UDP to TCP?
- Consumer VPNs like NordVPN vs Tailscale: one is a private exit to the internet, the other connects your own devices. How different are they underneath?

## Part 2 (about 14:43 to 14:52 BST, then parked)

### Why mesh, not hub (settled)

Qing's first answer: a hub means more traffic through one place. With prompting, settled: a hub is a **bottleneck**, it **adds latency** (every packet takes a detour through it), and it is a **single point of failure**, so all the reliability has to go into one box. Her analogy: a full mesh is like one flat LAN, where every device can talk to every other directly. This answers the "why mesh" question from part 1. See [why mesh, not hub](knowledge/why-mesh-not-hub.md).

### Control plane vs data plane (settled)

To build a tunnel to a peer, each device needs that peer's reachable public IP:port and its public key. Qing reasoned that nodes must advertise themselves, and that a freshly booted device has to ask a well-known central place to find the others. So:

- **Control plane:** Tailscale's coordination server. Hub-like: it hands out keys and addresses.
- **Data plane:** direct device-to-device tunnels, a mesh over the internet. Traffic does not go through the coordination server.
- Existing tunnels keep working if the coordination server goes down; only changes (new devices, new keys) wait for it.

Settled: she reasoned it herself. See [control plane vs data plane](knowledge/control-plane-vs-data-plane.md).

### Gap found: NAT and Layer 4 mechanics (not yet settled)

Qing didn't know that NAT tells apart several devices behind one public IP **by port** (NAPT, port translation), or how ports work at Layer 4. She knows what TCP and UDP are *for*; the gap was the mechanics. Covered, high level:

- A **port** picks which program on a machine gets the packet.
- The **4-tuple** (source IP, source port, destination IP, destination port) identifies one connection.
- **TCP:** reliable and ordered. Three-way handshake (SYN, SYN-ACK, ACK), sequence numbers and acknowledgements, retransmits anything lost.
- **UDP:** just ports, a length and a checksum. No handshake, no retransmission, no ordering.
- **Cost of reliability:** a retransmission waits at least a round trip, which can blow a voice call's ~50 ms latency budget. Late audio is useless, so real-time traffic prefers UDP.

### Parked at

> Laptop and phone, behind the same home router, both use source port 5000 to talk to Google. What does the router change so the replies can be told apart?

(Answer for next time: the router rewrites the source port and keeps a mapping table from public port back to device and original port.)

Then: why unsolicited inbound packets get dropped by NAT, and how hole punching gets around that. Still open: TCP-over-TCP meltdown, i.e. why VPNs prefer UDP.

### How Qing likes to learn

- High level first, then detail.
- Socratic: questions, so she reasons it out.
- When a prerequisite is missing, she asks to step back and fill it in first.
- When the topic shifts, park it and summarise where we got to.
