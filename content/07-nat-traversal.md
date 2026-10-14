# NAT traversal

> Draft for review. Not the finished page.

The previous chapter leaves both sides of a conversation in the same jam if both are behind NAT. A mapping exists only after an inside device sends a packet, and the router delivers an inbound packet only when a row fits. Each side is waiting for the other to have spoken first, and neither knows the public address and port the other's router would write down, because that port is chosen when the packet leaves, not before.

That is the deadlock UDP hole punching has to break. This chapter is about UDP, because that is what WireGuard sends. TCP hole punching is a different and less reliable procedure; [RFC 5128](https://www.rfc-editor.org/rfc/rfc5128) describes both, and the rest of this page does not need the TCP one.

## Ask someone who can see you

A machine behind NAT can learn its public mapping by sending a UDP packet to a server that is on the open internet, and reading what that server saw. The server does not see `192.168.1.10:5000`. It sees the public address and port the NAT just created, and it says so in the reply.

That exchange is the useful core of STUN, Session Traversal Utilities for NAT, specified in [RFC 8489](https://www.rfc-editor.org/rfc/rfc8489). Tailscale's [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works) describes it the same way: a "what is my endpoint from your point of view?" request, and an answer of the form "this is the address and port I saw."

The request has to leave from the same socket the real traffic will use. A socket here is just the local address and port a program sends from. Each socket gets its own mapping on the NAT. A STUN answer learned on some other socket is the public face of that other socket, and it is useless for the conversation you meant to have. The same post makes that point, and it is the sort of detail a simulator should not skip.

STUN creates the mapping as a side effect of asking. Until that packet went out, there was nothing to report.

## Tell the peer, then speak at once

The two peers now each know a public endpoint for themselves. They still cannot tell each other through the mapping, because each is about to send to an address it does not know yet. They exchange the endpoints through some path that is already open to both of them. [RFC 5128](https://www.rfc-editor.org/rfc/rfc5128) calls that helper a rendezvous server. In Tailscale the helper is the coordination server, with the DERP relays as a further side channel. Those are the next two chapters. For hole punching itself, any already-reachable helper is enough.

Then both peers send a UDP packet to the other's published public address and port, at about the same time.

Each NAT sees an outbound packet and installs a row toward that peer. Its filter now expects a reply from that peer's address and port. The packet coming the other way has exactly those addresses and ports, so it matches and is translated inward. The two packets do not have to form a neat question and answer. The filter from the previous chapter is what [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works) is talking about when it describes stateful firewalls: once some outbound packet has the right source and destination, an inbound packet that lines up is allowed through, even if the other side never received the packet you sent. A packet that arrives before the other NAT has a row is dropped. That is UDP hole punching, as [RFC 5128](https://www.rfc-editor.org/rfc/rfc5128) describes it.

## When the published port is a lie

STUN told each peer "you are this public address and port." Hole punching assumes the other peer, sending from somewhere else entirely, will hit that same port.

[RFC 4787](https://www.rfc-editor.org/rfc/rfc4787) splits NAT behaviour into two questions. The one that matters here is mapping behaviour: the inside device sends from one local address and port, and the NAT assigns a public port. Does that public port stay the same no matter which destination it talks to?

- **Endpoint-independent mapping.** Yes. Traffic from that local address and port leaves from one public address and port, whatever the destination. The port STUN saw is the port a peer should send to.
- **Endpoint-dependent mapping.** No. A different destination gets a different public port. [RFC 4787](https://www.rfc-editor.org/rfc/rfc4787) distinguishes address-dependent mapping (the public port changes with the destination address) from address-and-port-dependent mapping (it changes with the destination address and port). [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works) treats both as the hard case, and says the difference between them does not matter for traversal. The port you learned from STUN is not the port your peer must use.

"Endpoint" in that name means the destination. The mapping still depends on the source. If it did not, two devices behind the router would be indistinguishable, which is the problem the previous chapter exists to solve.

Filtering is the other question: once a row exists, who is allowed to send into it? [RFC 4787](https://www.rfc-editor.org/rfc/rfc4787) defines endpoint-independent, address-dependent, and address-and-port-dependent filtering. Talking to STUN has already created a row. On an endpoint-independent mapping, that is the same public port your peer would use. If the filter were also endpoint-independent, any internet host could send into that row, and the peer would not need you to send first. The Tailscale post says routers in the wild are overwhelmingly the strictest of the three, address-and-port-dependent filters, which accept a reply only from the remote you actually contacted. STUN is not your peer, so that row does not admit the peer. Each side still has to send to the other. The same post says that simultaneous send gets through all three filters. The filter is why both sides must send to each other, even after STUN. It is not why a correctly published port fails. Dependent mapping is why.

Older write-ups call these "cone" and "symmetric" NATs. The post includes a table from those names back to the RFC 4787 terms. The RFC terms are the ones to keep.

## Where it stops working

One hard NAT on the path is enough to break the simple version. The peer with the easy NAT does not know which public port to send to, so it cannot open its own filter toward the right place. The Tailscale post does not stop there. It describes opening many ports on the hard side and probing from the easy side, using the birthday paradox to make a collision likely, and it points at requirement 2 of [RFC 4787](https://www.rfc-editor.org/rfc/rfc4787) for why the public address is usually still trustworthy when the port is not. The probabilities live in that post. They do not need to be copied here. The shape of the failure is the point: the endpoint you can learn is not the endpoint you must reach.

Two hard NATs are worse. Each random probe also leaves from a random source port, so the search is for a matching pair of ports, not a single destination port. The same post says that search is much harder.

Some networks drop UDP altogether. There is then no STUN exchange and no hole to punch. The packets need a path that is allowed through, which is what the relay chapter is for.

A last failure, which shows up when two peers sit behind the same outer NAT. Carrier-grade NAT, which [RFC 6598](https://www.rfc-editor.org/rfc/rfc6598) makes room for with the shared range `100.64.0.0/10`, puts a second translator at the ISP, outside the home router. STUN reports the address on the far side of that outer translator. For two customers of the same ISP to use those addresses, the outer NAT has to hairpin: take a packet aimed at its own public address and turn it back inward. [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works) says many NATs do not hairpin, because they were built on the assumption that a packet for a public address always goes outward. If hairpinning fails and nothing else provides a path, the post's conclusion is a relay.

Port-mapping protocols (UPnP, NAT-PMP, PCP) are the other way out the post discusses: ask the NAT to open a public port on purpose. They are worth a sentence in the finished page and no more, because a network you do not administer, and a carrier NAT in particular, will not honour the request.

## Check yourself

1. Both laptops are behind NAT. Why does each one have to send a packet before either one can receive a packet from the other?
2. STUN says you are `203.0.113.5` port 4242. Under what kind of mapping is that the wrong port to give your peer, and why does sending from both sides not fix that by itself?

## Simulator

> **Simulator (not built yet).** Two home NATs, a laptop behind each, and a STUN server both can reach. Each laptop sends a STUN query and is shown the public address and port the server observed. They exchange those through a drawn side channel. Then both send UDP at the published endpoint, the mapping tables gain a row, and a following packet each way is translated in. A toggle switches one NAT to endpoint-dependent mapping: the port STUN recorded and the port created toward the peer are different, the peer's packet hits nothing, and the exchange fails in view. A second toggle drops UDP and leaves both sides with nowhere to send. The relay is the next chapter's sim, not this one.

## Sources

- [RFC 4787](https://www.rfc-editor.org/rfc/rfc4787), NAT behavioural requirements for unicast UDP. Mapping and filtering behaviours.
- [RFC 5128](https://www.rfc-editor.org/rfc/rfc5128), peer-to-peer communication across NATs, including UDP hole punching.
- [RFC 6598](https://www.rfc-editor.org/rfc/rfc6598), the shared address space used for carrier-grade NAT.
- [RFC 8489](https://www.rfc-editor.org/rfc/rfc8489), Session Traversal Utilities for NAT (STUN).
- Tailscale, [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works).
