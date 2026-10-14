# What a VPN is, and the kinds

> Outline only. Not the finished chapter. Beats for the prose Qing will write.

The definition to open with: a VPN is a network built in software on top of another network. Then refuse the idea that it is one product. Four dimensions sort the examples the page has already shown. Each dimension should be a sentence plus one familiar example, not a taxonomy lecture.

## The four dimensions

1. **What it carries.** Layer 2 (a VPWS wire, a VPLS LAN) or layer 3 (an L3VPN, an IP-in-IP tunnel, WireGuard).
2. **Its shape.** Point-to-point, hub-and-spoke, or mesh. Point at the hub-and-mesh chapter rather than arguing it here.
3. **What it rides on.** Provider labels, or IP and UDP across the public internet.
4. **Where privacy comes from.** Isolation (labels and VRFs; the provider still sees the inner packet) or encryption plus authentication (the inner packet is unreadable and bound to a key). A design can use one of these, or both. MPLS provider VPNs, as covered in the tunnelling chapter, use isolation alone.

After the four, place two worked examples on them so the reader sees the grid is usable:

- A provider L3VPN: carries layer 3, rides on labels, privacy from isolation. Shape depends on how the customer attaches; do not force it into "mesh".
- An internet tunnel between two gateways: carries layer 3, point-to-point, rides on IP (and often UDP), privacy from encryption if the tunnel actually encrypts. IP-in-IP by itself does not. The chapter should not let "tunnel" and "encrypted" collapse into each other.

## The consumer-VPN aside

Keep this short and about the job, not about a vendor's implementation.

A service such as NordVPN sells a private exit onto the internet. Your device builds a tunnel to the provider's gateway, and from there your packets go out to websites, which see the provider's address rather than your home address. Tailscale's usual job is different: it connects your own devices to each other, so a laptop can reach a machine you own.

Both are tunnels riding on the internet. The dimension that changes is what the tunnel is for (an exit, or a path to your own machines), and typically where it terminates. Do not name NordVPN's protocol, cipher, or logging policy. Those were not checked, and they are not the point. If a later rewrite wants them, look them up then.

The open question from the first session can stay open in one sentence: how different the two are underneath, once you ignore the job and look only at encapsulation and keys.

## Check yourself

1. Take an MPLS L3VPN and a WireGuard tunnel. For each, name the four dimensions.
2. Someone says "I use a VPN so websites don't see my home address." Which job is that, and which of the four dimensions does that sentence actually tell you?

## Simulator

> **Simulator (not built yet).** The same two sites drawn once. Four controls, one per dimension: payload layer, shape, underlay, and privacy. Changing a control redraws the packet (label versus outer IP header, ciphertext versus a readable inner header) and the path (one wire, a hub, or direct peers). A preset button labelled "private exit" should show a tunnel to a gateway that then forwards to the internet, next to a preset labelled "my own devices" that shows two machines with tunnels between them and no third-party exit.
