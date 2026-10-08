# 2026-10-08: first session

Socratic session, about 13:59 to 14:42 BST. High-level pass over routing, tunnelling and what a VPN is; the chapter order in the [learning path](index.md) came out of it.

## Already knew

- A solid layer-3 picture: packets hop router to router, each router deciding the next hop from its routing table.
- Longest-prefix match: when several routes match a destination, the most specific one wins. Answered without prompting.

## Recalled with prompting

- NAT at the edge (private home addresses rewritten to one public address).
- ARP, and that the layer-2 header is rewritten at every hop while the IP header carries on.
- MPLS: labels sitting at "layer 2.5", between the layer-2 and IP headers.
- Tunnelling as the general idea of wrapping one packet inside another.

## Corrections

- **The internet core has no default route.** Core routers carry the full BGP table, so there is no "send it upstream" fallback there. Default routes live at the edge.
- **A VPN is encapsulation, not NAT.** The inner packet is wrapped in a new outer header and unwrapped at the far end; nothing is rewritten in place.
- **"Private" in MPLS VPNs meant isolation**, not encryption. Each customer's traffic is kept apart by labels and VRFs; it is not encrypted.

## Settled

- [Longest-prefix match](knowledge/longest-prefix-match.md).

## Open questions

- Why is Tailscale a mesh rather than hub-and-spoke?
- Why do VPNs prefer UDP to TCP?
- Consumer VPNs like NordVPN vs Tailscale: one is a private exit to the internet, the other connects your own devices. How different are they underneath?
