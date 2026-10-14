# NAT and port translation

> Draft for review. Not the finished page.

A home router doing NAT takes packets from private addresses and sends them out from one public address. That is the whole story if one device is online. It stops being enough as soon as a second device is online too. Both replies come back to the same public address. Something else has to say which reply belongs to which device.

That something is the port number, in a form of NAT called NAPT, Network Address Port Translation. [RFC 2663](https://www.rfc-editor.org/rfc/rfc2663) defines it, and [RFC 3022](https://www.rfc-editor.org/rfc/rfc3022) describes the operation: many private hosts share one external address because the translator also rewrites the transport identifier, usually a TCP or UDP port.

Before that table will make sense, the port itself has to.

## A short stop at layer 4

The IP header says which machine a packet is for. It does not say which program. That is what the next header is for. TCP and UDP, the two layer-4 protocols you meet constantly, each start with a source port and a destination port. A port is a 16-bit number, so it runs from 0 to 65535. The destination port is how the receiving machine picks a listening program. The source port is how the sender's replies find their way back to the program that asked.

Write down the two addresses and the two ports and you have a 4-tuple: source IP, source port, destination IP, destination port. That names one flow between two programs. NAT also has to remember the protocol, because TCP port 5000 and UDP port 5000 are not the same flow. Add the protocol and the name is a 5-tuple. [RFC 4787](https://www.rfc-editor.org/rfc/rfc4787), which sets out how a NAT should behave for UDP, uses that 5-tuple.

TCP and UDP use those port fields very differently.

TCP ([RFC 9293](https://www.rfc-editor.org/rfc/rfc9293)) sets up a connection before data. The handshake is three segments: SYN, then SYN-ACK, then ACK. Each byte of data is numbered. The receiver acknowledges what it has got, and the sender transmits again anything that was not acknowledged. The far side delivers the bytes in order. The price of that repair is time. Discovering a loss and sending the data again takes at least one round trip. For a file, that is what you want. For a voice call, a chunk of audio that arrives after the moment it was meant to be played is useless, so the retransmission bought nothing.

UDP ([RFC 768](https://www.rfc-editor.org/rfc/rfc768)) does not make that promise. Its header is a source port, a destination port, a length, and a checksum. There is no handshake, no sequence number, and no retransmission. If a datagram is lost, it stays lost unless the application sends another one. That is why a tunnel that wants loss to stay loss, and a tunnel that must not stack a second copy of TCP's clocks, both end up on UDP. The WireGuard chapter is where that second point is argued. Here, UDP only has to be concrete: ports, and almost nothing else.

## Two devices, one public address

Take a laptop and a phone on the same home LAN. The laptop is `192.168.1.10`. The phone is `192.168.1.11`. Both open a connection to a website at `192.0.2.1`, port 443, and both happen to use source port 5000. Those are private addresses from [RFC 1918](https://www.rfc-editor.org/rfc/rfc1918) and a documentation address from [RFC 5737](https://www.rfc-editor.org/rfc/rfc5737). The port numbers are picked so the collision is obvious. A real phone would usually pick a random high port, and the collision would be rarer and no less real.

The home router's public address is `203.0.113.8`, another documentation address. If the router only rewrote the source IP, both packets would leave as `203.0.113.8` port 5000, going to `192.0.2.1` port 443. The website would answer both at `203.0.113.8` port 5000. The router, looking at the reply, would have two devices that fit and no way to choose.

So the router rewrites the source port as well, and writes down what it did. One possible table, with the external ports chosen for this telling:

| Inside | Outside | Remote |
|---|---|---|
| `192.168.1.10:5000` | `203.0.113.8:40001` | `192.0.2.1:443` |
| `192.168.1.11:5000` | `203.0.113.8:40002` | `192.0.2.1:443` |

The laptop's packet leaves as if `203.0.113.8` port 40001 had sent it. The phone's leaves as port 40002. The protocol, TCP in this case, is part of the row even though it is not drawn. The website never hears about `192.168.1.10`.

Rewriting the address and the port changes the TCP or UDP checksum, which covers both, so the router recomputes it. [RFC 3022](https://www.rfc-editor.org/rfc/rfc3022) includes that in the translation. ICMP is the other common case: a NAT of this kind translates the query identifier instead of a port. The chapter's story is easier to see with TCP and UDP, and those are the flows a VPN actually rides.

## The reply, and the packet that has no row

A reply arrives at `203.0.113.8` port 40001, from `192.0.2.1` port 443. The router finds that row, rewrites the destination to `192.168.1.10` port 5000, and puts the packet on the LAN. The laptop's stack delivers it to whichever program is using port 5000. The phone's reply, to port 40002, takes the other row. Same public IP, two mappings, two programs.

The table is filled in by packets that start inside. Until the laptop sends something, there is no row for it.

Now suppose a packet arrives at `203.0.113.8` on a port that is not in the table. The router has a public address and a port, and no inside device to give the packet to. It drops the packet. That is why an unsolicited inbound packet does not reach a laptop that has not spoken. There is nowhere to translate it to.

A row is also not an open door from the whole internet. [RFC 4787](https://www.rfc-editor.org/rfc/rfc4787) treats filtering as a separate decision from mapping: once a row exists, a NAT may still accept packets only from the remote address and port the inside device talked to. [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works) reports that this strict filtering is the common case on routers in the wild. The next chapter needs that distinction. For this one, the useful fact is smaller: inbound packets are delivered only where the table has a row that fits, and rows are created by outbound packets.

This is all still NAT. It is not a VPN. A VPN wraps the original packet in a new header and unwraps it at the far end. The router above edits the packet's own address and port, and keeps a note so it can edit the reply back.

## Check yourself

1. The laptop and the phone both send from source port 5000 to the same website. What does the router change on the way out, and what does it store so the two replies do not get mixed up?
2. A packet from the internet arrives at the public address on a port with no row in the table. What does the router do with it, and why is "deliver it to every inside device" not an option?

## Simulator

> **Simulator (not built yet).** One public address, a laptop and a phone, both sending to the same remote port from the same source port. Show the mapping table gaining a row as each packet leaves, with the source port rewritten to a distinct public port. Animate the two replies landing on the matching rows and coming out on the LAN at the right device and the original port. Then send a packet at an unused public port and show it discarded because no row exists. Separately, send a packet at a mapped port from a different remote address and show the row still there while the filter drops the packet. The table should be the whole interface. No tunnel, no keys.

## Sources

- [RFC 768](https://www.rfc-editor.org/rfc/rfc768), User Datagram Protocol.
- [RFC 1918](https://www.rfc-editor.org/rfc/rfc1918), private address space.
- [RFC 2663](https://www.rfc-editor.org/rfc/rfc2663), NAT terminology, including NAPT.
- [RFC 3022](https://www.rfc-editor.org/rfc/rfc3022), traditional NAT, including port translation and checksum adjustment.
- [RFC 4787](https://www.rfc-editor.org/rfc/rfc4787), NAT behavioural requirements for unicast UDP: the 5-tuple, mapping, and filtering.
- [RFC 5737](https://www.rfc-editor.org/rfc/rfc5737), the documentation address blocks used above.
- [RFC 9293](https://www.rfc-editor.org/rfc/rfc9293), Transmission Control Protocol.
- Tailscale, [How NAT traversal works](https://tailscale.com/blog/how-nat-traversal-works), for the report that address-and-port-dependent filtering is the common case in the wild. The RFC is the definition; the blog is the field observation.
