# VPNs and Tailscale: learning path

High level first, then detail. Socratic: questions first, corrections where needed. Order agreed by Qing 2026-10-08; it follows the page outline in [INTENT.md](../../INTENT.md).

| # | Chapter | Status |
|---|---------|--------|
| 1 | Normal IP routing: longest-prefix match, NAT at the edge, ARP and per-hop L2 rewrite, BGP / full table in the core, CDNs via DNS or anycast | **Started.** Covered at high level in [first session](2026-10-08-first-session.md). Settled: [longest-prefix match](knowledge/longest-prefix-match.md). |
| 2 | Tunnelling / encapsulation: MPLS provider VPNs (labels at L2.5, VPWS, VPLS, L3VPN with VRFs), then IP-in-IP / UDP over the internet | **Started.** Recalled with prompting in first session; not yet settled. |
| 3 | What a VPN is, and the kinds (carries L2/L3; point-to-point, hub-and-spoke, mesh; rides on labels or IP/UDP; privacy from isolation or crypto) | **Started.** Framework sketched in first session; consumer-VPN aside open. |
| 4 | Encryption and authentication: WireGuard vs IPsec/IKE and OpenVPN/TLS; UDP vs TCP | Not started. Open question: why UDP, not TCP. |
| 5 | Shape: hub vs mesh, and why Tailscale is a mesh | Not started. Open question: why mesh. |
| 6 | NAT traversal, including DERP relays | Not started. |
| 7 | Control plane: how everything gets configured | Not started. |

## Sessions

- [2026-10-08: first session](2026-10-08-first-session.md)

## Settled knowledge

- [Longest-prefix match](knowledge/longest-prefix-match.md)
