# Control plane vs data plane

A mesh VPN splits into two jobs:

- **Control plane:** how devices find each other and agree on keys. To build a tunnel, a device needs each peer's reachable public IP:port and public key. A freshly booted device can't know these, so it asks a well-known central place. In Tailscale that is the **coordination server**: hub-like, but it only carries configuration.
- **Data plane:** the actual traffic. It flows **directly device to device**, through encrypted tunnels over the internet, in a mesh. It does not pass through the coordination server.

Consequence: if the coordination server goes down, existing tunnels keep working. Only changes (a new device joining, keys rotating) have to wait for it to come back.

Settled 2026-10-08: Qing reasoned this herself in [part 2 of the first session](../2026-10-08-first-session.md#part-2-about-1443-to-1452-bst-then-parked).

## See also

- [Why mesh, not hub](why-mesh-not-hub.md)
- [Learning path](../index.md)
