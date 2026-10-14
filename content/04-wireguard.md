# Encryption and authentication: WireGuard

> Outline only. Not the finished chapter. Beats for the prose Qing will write. Crypto details should be taken from the WireGuard site and whitepaper when the prose is written, not filled in from memory beyond the list below.

WireGuard is the tunnel Tailscale's data plane is made of. The chapter should explain it as a small UDP tunnel with fixed cryptography, then say what that replaces, then answer why the outer transport is UDP.

## Beats to cover

- **What it is.** An open source VPN tunnel. It began in the Linux kernel and now runs on other operating systems as well. Tailscale's overview says their node software is built on the userspace implementation, wireguard-go. Cite [wireguard.com](https://www.wireguard.com/) and the [technical whitepaper](https://www.wireguard.com/papers/wireguard.pdf).
- **The picture from the conceptual overview.** You add a tunnel interface. An outgoing IP packet is encrypted to a peer's public key and sent as a UDP datagram to that peer's current endpoint. An incoming UDP datagram is decrypted and checked; if it is authentic and the inner source address is allowed for that peer, it is accepted on the interface.
- **Cryptokey routing, in one breath.** Each peer has a public key and a list of allowed addresses. On the way out, that list works like a routing table (which peer gets this destination). On the way in, it works like a check (is this peer allowed to send this source). That is the model's whole policy language. Key distribution is deliberately out of scope for WireGuard; the control-plane chapter is where Tailscale does that job.
- **Fixed choices, not a negotiation.** The site names the set: the Noise protocol framework, Curve25519, ChaCha20, Poly1305, BLAKE2, SipHash24, HKDF. The whitepaper is the place to say how they fit together (handshake, then an authentication tag on data packets). Contrast, briefly and without a feature matrix, with IPsec and IKE (a lot of options, negotiated) and with OpenVPN over TLS (a userspace TLS session). The point of the contrast is why a mesh of many peers wants a small, fixed tunnel. Do not claim WireGuard is universally faster; the site claims it aims to be, and that is enough.
- **Why the outer transport is UDP.** Two reasons, and they should not be blurred.
  - WireGuard itself encapsulates IP packets in UDP. There is no TCP mode in WireGuard. Tailscale's overview says DERP is what adds a TCP-friendly path, and that this is outside WireGuard proper. Point at the relay chapter.
  - If the tunnel were a TCP connection, and the inner traffic were also TCP, the two retransmission clocks fight. Olaf Titz's note [Why TCP Over TCP Is A Bad Idea](https://web.archive.org/web/20150901113742/http://sites.inka.de/bigred/devel/tcp-tcp.html) is the source to explain from: when the outer TCP loses a segment it backs off, the inner TCP does not get its acknowledgement, and the inner TCP retransmits faster than the outer one can drain. This was still an open question after the first session. The finished chapter should explain that mechanism, not stop at "UDP is lighter".
  - A separate, smaller point from the layer-4 primer: a tunnel that retransmits on its own waits at least a round trip to recover a loss. For voice, a packet that arrives late is useless. That argues for UDP when the *application* wants loss to stay loss. It is not the same argument as TCP-over-TCP meltdown. Say both, labelled.

## Check yourself

1. A UDP datagram arrives on a WireGuard interface and decrypts cleanly as coming from peer A. The inner source address is not in peer A's allowed list. What happens to it?
2. Why is a TCP tunnel that carries TCP sessions a worse idea than it first sounds? What, specifically, do the two TCP stacks both try to do after one lost segment?

## Simulator

> **Simulator (not built yet).** One packet enters a `wg0` interface. The reader watches it get encrypted to a peer public key and leave as a UDP datagram toward that peer's endpoint. On the far side it is accepted only if the authentication tag checks and the inner source is in the allowed list. A button flips a bit in the UDP payload and the far side drops it. A second toggle, labelled as a thought experiment rather than as WireGuard, wraps the same inner TCP session in an outer TCP connection and shows the outer timeout stretching while the inner stack emits extra retransmissions. That second scene should be visibly not WireGuard, so the sim does not imply WireGuard can run over TCP.
