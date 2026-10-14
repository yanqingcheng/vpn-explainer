# Tunnelling and encapsulation

> Outline only. Not the finished chapter. Beats for the prose Qing will write.

Start from provider MPLS, which is the familiar case, and only then cross to wrapping an IP packet in another IP packet on the public internet. The correction to land, in plain words: encapsulation wraps a packet in a new outer header and unwraps it at the far end. NAT rewrites addresses in place. They are different operations.

## Beats to cover

- **Where an MPLS label sits.** Between the layer-2 header and the IP header. Calling it layer 2.5 is fine if the chapter says it is a teaching nickname, not a layer in the formal stack. Cite the MPLS architecture, [RFC 3031](https://www.rfc-editor.org/rfc/rfc3031).
- **Three provider VPN services, by what they carry and their shape.**
  - VPWS: a point-to-point layer-2 service. One attachment circuit bound to one pseudowire. [RFC 4664](https://www.rfc-editor.org/rfc/rfc4664).
  - VPLS: a multipoint layer-2 service that emulates a LAN across the provider. Same framework RFC; signalling detail can point at [RFC 4761](https://www.rfc-editor.org/rfc/rfc4761) or [RFC 4762](https://www.rfc-editor.org/rfc/rfc4762) without teaching either one.
  - L3VPN: the customer routes live in a VRF on the provider edge, and the packet is labelled across the core. [RFC 4364](https://www.rfc-editor.org/rfc/rfc4364).
- **Privacy in that world is isolation.** Labels and VRFs keep one customer's packets out of another's forwarding table. The provider can still see the customer's packet. Nothing in this design encrypts it. Say that directly; it is the distinction the rest of the page builds on.
- **The same idea over the internet.** The original packet is left intact and placed behind a new outer header. For IP-in-IP, cite [RFC 2003](https://www.rfc-editor.org/rfc/rfc2003): outer source is the home side's public address, outer destination is the far gateway's public address. The gateway strips the outer header and routes the inner packet as if it had arrived locally. UDP encapsulation is the same wrapping with a UDP header as well, which is the form WireGuard uses; point forward rather than inventing a generic RFC for "tunnels over UDP".
- **What the core sees.** On the MPLS path, core routers forward on the label. On the IP path, internet routers forward on the outer header only. The inner addresses are payload to them.

## Check yourself

1. A packet goes from a home laptop to a host on an office LAN, wrapped in an outer IP header. Which addresses do the routers on the internet look at, and what does the office gateway do before the inner packet can be delivered?
2. In an MPLS L3VPN, what keeps two customers apart, and what does that mechanism not do?

## Simulator

> **Simulator (not built yet).** Two short scenes of the same inner packet. In the first, it crosses three provider routers with an MPLS label: the edge pushes a label, the middle swaps it, the far edge pops it, and the inner IP header is unchanged the whole way. In the second, the inner packet sits behind an outer IP header (and, on a toggle, a UDP header) from a home public address to an office gateway; internet hops forward on the outer header only; the gateway strips it and routes the inner packet onto the office LAN. A side-by-side with NAT, where the source address itself changes, would make the wrap-versus-rewrite difference visible.
