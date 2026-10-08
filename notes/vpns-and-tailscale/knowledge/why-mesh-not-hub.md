# Why mesh, not hub

In a **hub-and-spoke** VPN, every packet between two devices goes through a central hub. In a **mesh**, devices talk to each other directly.

Why a mesh (for the data):

- **No bottleneck.** A hub has to carry everyone's traffic.
- **Lower latency.** No detour through the hub; packets take the direct path.
- **No single point of failure.** If the hub dies, everything dies, so all the reliability has to go into one box. In a mesh, one device failing only affects that device.

Analogy: a full mesh is like one flat LAN, where any device can reach any other directly.

A mesh still needs something central to help devices find each other; see [control plane vs data plane](control-plane-vs-data-plane.md).

Settled 2026-10-08 with prompting, in [part 2 of the first session](../2026-10-08-first-session.md#part-2-about-1443-to-1452-bst-then-parked).

## See also

- [Learning path](../index.md)
