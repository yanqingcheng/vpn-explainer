# VPNs and Tailscale: learning path

High level first, then detail. Socratic: questions first, corrections where needed. Order agreed by Qing 2026-10-08; it follows the page outline in [INTENT.md](../../INTENT.md).

| # | Chapter | Status |
|---|---------|--------|
| 1 | Normal IP routing: longest-prefix match, NAT at the edge, ARP and per-hop L2 rewrite, BGP / full table in the core, CDNs via DNS or anycast | **Started.** Covered at high level in [first session](2026-10-08-first-session.md). Settled: [longest-prefix match](knowledge/longest-prefix-match.md). |
| 2 | Tunnelling / encapsulation: MPLS provider VPNs (labels at L2.5, VPWS, VPLS, L3VPN with VRFs), then IP-in-IP / UDP over the internet | **Started.** Recalled with prompting in first session; not yet settled. |
| 3 | What a VPN is, and the kinds (carries L2/L3; point-to-point, hub-and-spoke, mesh; rides on labels or IP/UDP; privacy from isolation or crypto) | **Started.** Framework sketched in first session; consumer-VPN aside open. |
| 4 | Encryption and authentication: WireGuard vs IPsec/IKE and OpenVPN/TLS; UDP vs TCP | Not started. Open question: why UDP, not TCP (TCP-over-TCP meltdown). |
| 5 | Shape: hub vs mesh, and why Tailscale is a mesh | **Settled** in [first session, part 2](2026-10-08-first-session.md#part-2-about-1443-to-1452-bst-then-parked): [why mesh, not hub](knowledge/why-mesh-not-hub.md). |
| 6 | NAT traversal, including DERP relays | **Started.** Gap found in NAT port translation (NAPT); parked there. |
| 7 | Control plane: how everything gets configured | **Started.** Settled: [control plane vs data plane](knowledge/control-plane-vs-data-plane.md). Detail (keys, ACLs, node maps) not yet. |

## Next session

1. **NAT port translation (NAPT).** Pick up the parked question: laptop and phone both use source port 5000 to Google; what does the router change so replies can be told apart?
2. **NAT traversal.** Why unsolicited inbound packets are dropped; hole punching.
3. **DERP relays.** What happens when hole punching fails.

### Layer 4 prerequisites

Introduced in part 2, not yet settled: ports pick the program; the 4-tuple identifies a connection; TCP (handshake, sequence numbers, acks, retransmits) vs UDP (ports, length, checksum only); why retransmission hurts real-time traffic. Check these quickly before NAPT.

## Sessions

- [2026-10-08: first session](2026-10-08-first-session.md) (two parts; parked at NAT port translation)

## Settled knowledge

- [Longest-prefix match](knowledge/longest-prefix-match.md)
- [Why mesh, not hub](knowledge/why-mesh-not-hub.md)
- [Control plane vs data plane](knowledge/control-plane-vs-data-plane.md)
