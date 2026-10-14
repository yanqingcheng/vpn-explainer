# DERP relays

> Draft for review. Not the finished page.

Hole punching is an attempt to get a direct UDP path. Sometimes there is no such path. One or both NATs are the hard kind from the previous chapter, the network drops UDP, or two peers behind the same carrier NAT cannot hairpin. The two devices can still both reach the open internet. A relay is a machine there that both are willing to send to, which forwards what it receives.

In the usual standard procedure for this, ICE, that job is TURN, Traversal Using Relays around NAT. ICE is the protocol WebRTC and others use to try a direct path and fall back to a relay. You do not need the rest of it here. Tailscale did not use TURN. [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works) gives the reasons they published: TURN is an unpleasant protocol to implement, and there is no open population of TURN servers to interoperate with, so a private relay protocol loses nothing. They built their own, and called it DERP.

The March 2020 overview [How Tailscale works](https://tailscale.com/blog/how-tailscale-works) and Tailscale's current [DERP servers](https://tailscale.com/docs/reference/derp-servers) docs expand that name as Designated Encrypted Relay for Packets. The NAT traversal post expands the same four letters as Detour Encrypted Routing Protocol. The behaviour described is the same. This chapter follows the overview and the current docs for the name.

## What the relay is allowed to see

A DERP server forwards packets from one Tailscale node to another. It decides where to send them from the destination's public key. The payload it forwards is already a WireGuard packet.

The private key that could decrypt that packet is generated on the node and stays there. [How Tailscale works](https://tailscale.com/blog/how-tailscale-works) says the relay "blindly forwards already-encrypted traffic from one node to another", and compares it to any other router on the path, using a slightly stricter protocol to prevent abuse. The current [DERP servers](https://tailscale.com/docs/reference/derp-servers) docs say the same thing: because private keys never leave the device that generated them, the relay cannot decrypt the traffic. [Tailscale encryption](https://tailscale.com/docs/concepts/tailscale-encryption) repeats it.

So a relay that is carrying your session can see an encrypted packet and which public key it is addressed to. It cannot read the inner packet. It is not a VPN concentrator that terminates the tunnel and starts a new one. The encryption runs end to end, relay or no relay.

The overview describes the relays as HTTPS streams, in the role TURN plays, using WireGuard keys rather than TURN's own arrangements. The NAT traversal post says the relay protocol runs over HTTP. Those two public descriptions do not use the same word, so this draft will not pick one. What both support is that the path is the kind of path web traffic uses, which is why it still gets out of networks that block UDP, and that the thing inside it is an encrypted WireGuard packet. WireGuard itself has no TCP mode. The overview is explicit that the TCP-friendly path is DERP, in the Tailscale node software, not in WireGuard.

## Fallback, and the path that gets you off the fallback

DERP is the fallback when a direct path cannot be made. It is also part of how a direct path gets made. [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works) says Tailscale uses the coordination server and the DERP fleet as the side channel for the endpoints in the previous chapter, and that every connection starts with DERP already selected. User traffic can flow through that relay immediately, while the two nodes try UDP in parallel. If a direct path comes up, the connection moves to it. If it never does, the relay keeps carrying the packets.

That is a different shape from "try hole punching for a while, and only then think of a relay." In that write-up the relay is there from the start, and the direct path is an upgrade.

Two details from the current docs belong beside that, so this does not describe an older Tailscale than the one a reader will install.

A DERP server carries more than the user's packets. The [DERP servers](https://tailscale.com/docs/reference/derp-servers) docs say it forwards DISCO packets, which are Tailscale's discovery messages, and encrypted WireGuard packets. Most of the time it is the discovery messages, while the direct path is being arranged. When the direct path never appears, it is the WireGuard packets.

A tailnet, the set of devices in one Tailscale network, can also run its own relay. The [connection types](https://tailscale.com/docs/reference/connection-types) docs say that if both devices can reach a peer relay on the tailnet, Tailscale tries that before a DERP server. Read with the earlier post, the order is: a peer relay when both sides have one, otherwise DERP from the start, and a direct UDP path later if hole punching finds one. The teaching point is the same in either case. The box in the middle is forwarding ciphertext.

Direct is still the path you want when you can get it. A relay is an extra hop, and it shares its capacity with everyone else using it. Those costs apply to the pairs that are still on the relay, and only to those pairs.

## Check yourself

1. Your direct UDP path has failed and your packets are going through a DERP server. What can that server see, and what can it not see?
2. Why can a Tailscale session already be carrying traffic before hole punching has succeeded?

## Simulator

> **Simulator (not built yet).** Two nodes that cannot punch a direct hole, and one DERP server both can reach over a web-like path. Packets flow node, relay, node. The packet drawn on the relay's hop should be an opaque blob labelled with the destination public key, and the inner IP packet should become visible only after the far node decrypts it. A button then "opens" a direct UDP path: new packets take the direct hop, the relay goes quiet, and the blobs the relay already forwarded stay opaque. Nothing in the sim should hand the relay a private key.

## Sources

- Tailscale, [How Tailscale works](https://tailscale.com/blog/how-tailscale-works). DERP as Designated Encrypted Relay for Packets; the relay forwards ciphertext; private keys stay on the node.
- Tailscale, [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works). Why not TURN; DERP as data fallback and as the side channel; connections start on DERP and upgrade.
- Tailscale, [DERP servers](https://tailscale.com/docs/reference/derp-servers). Current role, discovery packets versus data packets, and the claim that the relay cannot decrypt.
- Tailscale, [Connection types](https://tailscale.com/docs/reference/connection-types). Peer relay, when one exists, before DERP.
- Tailscale, [Tailscale encryption](https://tailscale.com/docs/concepts/tailscale-encryption). Relays handle packets that are already encrypted.
