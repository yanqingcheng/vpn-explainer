# Draft chapters

These are draft reader-facing chapters for the teaching page in [INTENT.md](../INTENT.md). They are here for Qing to review and rewrite. They are not the finished page, and they are not settled knowledge. What Qing has endorsed lives under [notes/vpns-and-tailscale/](../notes/vpns-and-tailscale/index.md).

Nothing here is wired to a simulator. Each chapter ends with a placeholder and a one-paragraph idea for the Go simulator that will eventually sit there.

## Order

The order is the page outline. The later chapters are full drafts. The earlier ones are outlines only.

| File | What it is |
|---|---|
| [01-ip-routing.md](01-ip-routing.md) | Outline |
| [02-tunnelling.md](02-tunnelling.md) | Outline |
| [03-what-a-vpn-is.md](03-what-a-vpn-is.md) | Outline |
| [04-wireguard.md](04-wireguard.md) | Outline |
| [05-hub-and-mesh.md](05-hub-and-mesh.md) | Outline |
| [06-nat-port-translation.md](06-nat-port-translation.md) | Full draft |
| [07-nat-traversal.md](07-nat-traversal.md) | Full draft |
| [08-derp-relays.md](08-derp-relays.md) | Full draft |
| [09-control-plane.md](09-control-plane.md) | Full draft |

## Points to check while rewriting

- **DERP's expansion.** [How Tailscale works](https://tailscale.com/blog/how-tailscale-works) (March 2020) and the current [DERP servers](https://tailscale.com/docs/reference/derp-servers) docs expand DERP as Designated Encrypted Relay for Packets. [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works) expands it as Detour Encrypted Routing Protocol, and once in that post as Detoured Encrypted Routing Protocol. The draft uses the overview and the current docs, and says so in the sources.
- **Peer relays.** Current Tailscale docs say that if a tailnet has a peer relay both sides can use, that is tried before a DERP server. The DERP chapter includes one sentence on this so it does not claim DERP is the only fallback.
- **A coordination outage.** The session notes say existing tunnels keep working and only changes wait. Tailscale's current page [What happens if the coordination server is down?](https://tailscale.com/docs/reference/coordination-server-down) agrees about traffic and cached policy, and adds limits: no new connections, no key refresh or revocation, no policy updates, and devices drift apart as locally stored keys expire. The draft follows that page. Worth keeping the expiry limit so the chapter does not promise that an outage is invisible.
- **4-tuple and 5-tuple.** The session used the 4-tuple (two addresses, two ports). [RFC 4787](https://www.rfc-editor.org/rfc/rfc4787) includes the protocol as well. The NAT chapter teaches both.
- **Voice latency.** The session mentioned a ~50 ms budget. The draft does not repeat that number. It only says a retransmission waits at least a round trip, which is too late for audio that had to arrive now.
- **NordVPN.** The "what a VPN is" outline treats it as an example of a private exit onto the internet. It does not name NordVPN's protocol.
- **Example addresses.** The NAT chapter uses documentation addresses from [RFC 5737](https://www.rfc-editor.org/rfc/rfc5737) and private addresses from [RFC 1918](https://www.rfc-editor.org/rfc/rfc1918). The port numbers are chosen for the example.
- **One key pair.** The control-plane draft follows [How Tailscale works](https://tailscale.com/blog/how-tailscale-works), which describes a single key pair per node. Tailscale's later [key-management post](https://tailscale.com/blog/tailscale-key-management) distinguishes more than one locally generated key. The draft does not open that. Private keys still stay on the device in both accounts.
