# How does a VPN work: intent

Affirmed by Qing, 2026-10-08. Edit this file first if intent changes, then change code. Lines marked *proposed* are suggestions Qing has not confirmed yet.

## Why

A teaching page that starts from what a telecoms/networking person already knows and builds up to how a modern mesh VPN like Tailscale really works, with a live simulator at each step. And, in Qing's words: **"I want to learn Go."**

## Goals

1. **Public-quality explainer.** Good enough for a general audience to read and share. Accuracy matters more than polish.
2. **Start from what Qing already knows.** Qing has a telecoms/networking background. Each step builds on that rather than starting from zero.
3. **Understand how Tailscale works.** By the end, Qing can explain a tailnet end to end: WireGuard tunnels between devices, the coordination server / control plane (keys, node maps, ACLs), NAT traversal, DERP relays when a direct path fails, and how they fit together.
4. **Learn Go.** A goal in its own right, not a side effect. Qing writes a real share of the Go herself.

## Audience

Curious non-specialists who use VPNs but don't know what they do, plus technical readers who will check the details. The page should not lose the first group or get anything wrong for the second.

## Outline (order agreed by Qing 2026-10-08, may still change)

1. **Normal IP routing.** Longest-prefix match; NAT at the edge; ARP, with the layer-2 header rewritten at every hop; BGP and the full routing table in the core; briefly, how CDNs steer you via DNS or anycast.
2. **Tunnelling / encapsulation.** First through provider MPLS VPNs: labels at "layer 2.5", VPWS, VPLS, and L3VPN with VRFs. Private by isolation, with no encryption. Then IP-in-IP / UDP over the internet: the outer header goes from the home public IP to the office gateway's public IP; the gateway strips it off and routes the inner packet.
3. **What a VPN is, and the kinds.** A network made in software on top of another network. Four dimensions: what it carries (layer 2 or layer 3); its shape (point-to-point, hub-and-spoke, mesh); what it rides on (provider labels, or IP/UDP over the internet); where privacy comes from (isolation, or encryption plus authentication). Aside: consumer VPNs like NordVPN (a private exit to the internet) vs Tailscale (connecting your own devices).
4. **Encryption and authentication.** WireGuard: open source, in the Linux kernel, small, fixed crypto choices, one handshake, an authentication tag on every packet. Compared with IPsec/IKE and OpenVPN/TLS. Why VPNs prefer UDP to TCP.
5. **Shape: hub vs mesh.** Tailscale is a mesh, and why.
6. **NAT traversal**, including DERP relays when a direct path fails.
7. **The control plane.** How everything gets configured: keys, who can reach whom, and how each device learns about the others.

Each step gets one short explanation plus one live simulator.

## Simulators

- Written in **Go**, compiled to **WebAssembly**, running in the browser. No server.
- One small sim per step (for example: send a packet across a few routers and watch the routing table decide each hop).
- The sim code doubles as Qing's Go practice, so it should read like ordinary, idiomatic Go, not WASM tricks.

## How Qing learns

- **High level first, then detail.** Get the whole picture of a step before going deep on any part of it.
- **Socratic.** Mostly questions, not lectures: Qing reasons her way to the answer from what she already knows, with corrections where she goes wrong.
- **Propose, don't impose.** A learning path is proposed for each step; Qing endorses or reshapes it.
- **Her pace, her curiosity.** Tangents are fine. If it feels like a slog, change the approach.
- **Challenge understanding.** When Qing says she gets something, test it (explain it back, predict what the sim will do, write the next function). Find gaps early.
- **Qing writes the Go.** Each sim's skeleton and tests are set up for her; she writes the core logic as guided exercises or pairing, then it gets reviewed. Generated code is for scaffolding, not for the parts she is meant to learn.
- **Keep what stuck.** Working notes can be messy; only what Qing has shown or endorsed she understands is written up as settled knowledge.
- **Notes live in this repo**, under [`notes/`](notes/README.md). They are public along with everything else here.

## Where it lives

The learning section of Qing's Workshop (qingsworkshop.com) once it's complete. Until then it stays in this repo.

## First thin version (*proposed*, default unless Qing changes it)

Step 1 only: normal IP routing, with one Go/WASM simulator, written mostly by Qing.

## Out of scope

- A real VPN or anything that touches real networks. These are teaching simulations.
- Editing qingsworkshop.com from this repo. The finished page is handed over to the site.

## Open questions for Qing

- Is the thin first version right (step 1, normal IP routing, one sim)?
- How much of the Go does she want to write herself (all the core sim logic, or a mix)?
