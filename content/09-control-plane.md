# The control plane

> Draft for review. Not the finished page.

A WireGuard tunnel needs two things about the peer before the first data packet: that peer's public key, and a place to send UDP, which might be a public address and port or a relay. A device that has just come up has neither. Addresses change, and keys are not something you want to type into every laptop by hand. Each device has to ask a place both of them can already find.

In Tailscale that place is the coordination server. [How Tailscale works](https://tailscale.com/blog/how-tailscale-works) calls it a shared drop box for public keys. The same post draws the shape this page has been building toward. The control plane is hub-and-spoke. The data plane is a mesh. The control plane "carries virtually no traffic. It just exchanges a few tiny encryption keys and sets policies."

## What gets handed out

The sequence in that post, pared down:

1. The node generates its own key pair, and keeps the private key on the device.
2. It contacts the coordination server and leaves its public key, where it can currently be reached, and which network it belongs to.
3. It downloads the public keys and addresses of the peers it is allowed to know about.
4. It configures WireGuard with those public keys.

The private key does not leave the node. The post is firm about why. The private key is what would let someone else impersonate that node in a WireGuard handshake. Because it stays put, only the two nodes in a session can encrypt or decrypt that session. The relay chapter depends on this sentence. A DERP server is one of the machines that forwards the result, and it is not one of the two.

"The peers in that network" is already a policy decision. The coordination server does not give every node every key. [How Tailscale works](https://tailscale.com/blog/how-tailscale-works) says a node is given the public keys of the nodes that are supposed to connect to it, and that without the right public key a stranger's packet cannot be decrypted. The current [control and data planes](https://tailscale.com/docs/concepts/control-data-planes) docs say the same thing: a device learns the public keys of the devices and resources it is allowed to reach, and does not learn the others. Policy is stored on the coordination server and enforced on each node, because in a mesh there is no concentrator the packets all pass through. The rules themselves are a later detail. The shape is the point. Who you can reach is decided centrally, and applied at the device.

The server also has to keep passing endpoints, not just keys. A laptop moves networks and its public address changes. The [control and data planes](https://tailscale.com/docs/concepts/control-data-planes) docs list this under connection coordination: peer discovery, the endpoint information NAT traversal needs, and which DERP region to use when a direct path is not up. Keys without a current endpoint would build a tunnel with nowhere to send.

Login is not a password database on that server. [How Tailscale works](https://tailscale.com/blog/how-tailscale-works) says Tailscale hands user authentication to an existing identity provider, over OAuth 2.0, OpenID Connect, or SAML. The coordination server still decides which public keys belong to which network, and which of those keys each node is allowed to see.

## What survives when the hub goes away

The data path does not go through the coordination server. Packets go device to device once a direct path is up. Until then, and whenever hole punching never succeeds, they go by a relay, as the previous chapter described. Either way, the coordination server is not forwarding them.

Tailscale's page [What happens if the coordination server is down?](https://tailscale.com/docs/reference/coordination-server-down) states the consequence. Devices keep the keys locally, and they keep a cached copy of the firewall rules, so tunnels that are already up keep carrying traffic and the cached policy keeps being applied. The [control and data planes](https://tailscale.com/docs/concepts/control-data-planes) note says the same: pre-established connections and cached policy continue, because traffic is not routed through the coordination service.

The outage is still an outage for anything that has to change. From those same two pages: new users and devices cannot be added, new connections are not established, keys cannot be refreshed or exchanged, keys cannot be revoked, and policy cannot be updated. Key expiry is per device, based on when that device last authenticated, so if the outage lasts long enough that a key expires, that device gradually loses the others. "Existing tunnels survive" is true. "Nothing happens" is not.

This is the mesh from the earlier chapter with its remaining hub named. The hub introduces the peers and tells them the rules. It is not on the path of the packets. While it is unreachable, the introductions it has already made continue, and it makes no new ones.

## Check yourself

1. Of these, which ones pass through the coordination server: a file you copy to another of your machines, your peer's public key, the packets of a tunnel that is already up?
2. The coordination server has been unreachable for a minute. What happens to a tunnel that was already carrying traffic? What can a device that has never fetched its peers' public keys and endpoints not do?

## Simulator

> **Simulator (not built yet).** Three nodes and a coordination server. Each node publishes a public key and an endpoint; the server's table is visible; each node receives only the peers it is allowed to know and draws a direct tunnel to them. User packets flow on those tunnels and do not touch the server. A "coordination down" switch freezes the table: packets already in a tunnel keep moving, and a fourth node that boots during the outage gets no keys and builds no tunnel. A separate "key expired" mark on one node, available only after the outage has been on, drops that node's tunnels, so expiry is a different event from the outage itself. No real cryptography. A labelled public key and a padlock on the packet are enough.

## Sources

- Tailscale, [How Tailscale works](https://tailscale.com/blog/how-tailscale-works). Coordination server as a drop box for public keys; control plane hub-and-spoke, data plane a mesh; private keys stay on the node; authentication handed to an identity provider.
- Tailscale, [Control and data planes](https://tailscale.com/docs/concepts/control-data-planes). What the coordination service distributes, and what continues when it is unavailable.
- Tailscale, [What happens if the coordination server is down?](https://tailscale.com/docs/reference/coordination-server-down). Existing traffic and cached policy continue; new devices, new connections, key refresh, revocation, and policy updates do not; keys expire per device.
