# Normal IP routing

> Outline only. Not the finished chapter. Beats for the prose Qing will write.

This chapter starts from a picture the reader may already have: a packet hops from router to router, and each router chooses the next hop by looking up the destination in its own table. It should stay at that altitude, then name the few mechanisms that make the public internet behave. Port translation gets its own chapter later. Here, NAT is only "the home router rewrites a private source address before the packet leaves".

## Beats to cover

- **Longest-prefix match.** Several routes may match. The longest prefix wins. For destination `10.1.2.3`, a table holding `10.0.0.0/8`, `10.1.0.0/16` and `10.1.2.0/24` picks `10.1.2.0/24`. A default route, `0.0.0.0/0`, matches everything, so it is used only when nothing more specific does. A more specific route can override a broader one without deleting it. This is settled in [longest-prefix match](../notes/vpns-and-tailscale/knowledge/longest-prefix-match.md).
- **Edge and core.** At the edge, a default route points upward. In the core, routers carry a full table, learned from neighbours with BGP ([RFC 4271](https://www.rfc-editor.org/rfc/rfc4271)). Do not turn this into a BGP tutorial. The point is where the detailed knowledge lives.
- **NAT at the boundary, one sentence.** Private source addresses ([RFC 1918](https://www.rfc-editor.org/rfc/rfc1918)) are rewritten to a public address as the packet leaves home. Say that several devices share that public address, and send the reader to the port-translation chapter for how the replies are told apart.
- **ARP and the layer-2 header.** The IP header is what the routers forward. The layer-2 header is rewritten at every hop: the router resolves the next hop (ARP on Ethernet, [RFC 826](https://www.rfc-editor.org/rfc/rfc826)) and puts that on the wire. A reader who already thinks in headers should see the IP header travel and the link header get replaced.
- **How a CDN steers you, briefly.** Two mechanisms only: DNS handing back a nearby address, and anycast (the same address announced from many places, so routing delivers you to a nearby one). No vendor tour.

## Check yourself

The finished chapter should end with questions in this spirit:

1. A router has `10.0.0.0/8`, `10.1.0.0/16` and `0.0.0.0/0`. A packet is destined for `10.1.2.3`. Which route does it use, and why does the default route not win?
2. Which header is rewritten at every hop, and which one still names the original destination when the packet arrives?

## Simulator

> **Simulator (not built yet).** A short row of routers, each with a small table. The reader sends one packet and watches each hop choose a next router by longest prefix. One table should contain both a default route and a more specific route, so the more specific one wins in view. A second packet, to an address nobody has a specific route for, should follow the default route off the edge. No NAT table in this sim; that belongs to the port-translation chapter.
