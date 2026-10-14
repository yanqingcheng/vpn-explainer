# Shape: hub and mesh

> Outline only. Not the finished chapter. The conclusion is settled in [why mesh, not hub](../notes/vpns-and-tailscale/knowledge/why-mesh-not-hub.md). The prose still has to be written for a reader who was not in the session.

## Beats to cover

- **Hub-and-spoke.** Every packet between two devices goes through a central hub. This is the usual way people first build a VPN, including a WireGuard one: the hub has a stable address and an open port, and everyone else dials it. [How Tailscale works](https://tailscale.com/blog/how-tailscale-works) draws this and explains why the bookkeeping is smaller.
- **Mesh.** Devices that need to talk do so directly. Tailscale's data plane is this shape.
- **Three costs of putting the data through a hub.** These are the settled reasons:
  - It is a bottleneck. Every pair's traffic has to fit through one box.
  - It adds latency. Packets take a detour. The blog's illustration is a person and a server in the same city whose packets hairpin through a concentrator on another coast. Paraphrase that; do not depend on the figure.
  - It is a single point of failure. If the hub dies, every pair dies with it, so all the reliability has to be built into one box. In a mesh, one device failing takes out that device's conversations.
- **The LAN analogy.** A full mesh feels like one flat LAN: any device can reach any other directly. Useful, and it should stay an analogy. A mesh over the internet still has NAT, keys, and failure in a way a switched LAN does not. The later chapters are that gap.
- **The hub does not disappear.** Something central still helps devices find each other and hands out keys. That something carries almost no user traffic. Say that here in two sentences and leave the mechanism to the control-plane chapter, so this chapter does not pretend a mesh has no shared infrastructure.
- **Why people still choose a hub.** One honest paragraph: a hub is easier to configure by hand (the blog's counting example: 10 fully connected nodes means 90 peer endpoints to keep updated), and it is a natural place to put a firewall. Tailscale's answer to the firewall problem is in the control-plane chapter (policy distributed, enforced on each node). Do not argue it fully here.

## Check yourself

1. Name the three costs of sending all data through a hub.
2. If a central server only introduces the peers, and the packets then go direct, which of those three costs still applies to the data path?

## Simulator

> **Simulator (not built yet).** Four devices. A toggle selects hub-and-spoke or mesh. The reader sends a packet between two of them and sees the path: via the hub, or direct. A "fail the hub" button drops every spoke-to-spoke flow in the hub mode, and drops only the failed device's own flows in the mesh mode. A hop counter on each packet makes the detour visible without any real geography. No keys and no NAT in this sim.
