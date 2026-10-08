# Longest-prefix match

When a router looks up a destination IP address, several routes in its table may match. It uses the one with the **longest prefix**, the most specific match.

Example: for destination `10.1.2.3`, a table holding `10.0.0.0/8`, `10.1.0.0/16` and `10.1.2.0/24` picks `10.1.2.0/24`. A default route (`0.0.0.0/0`) matches everything, so it is only used when nothing more specific does.

This is why a more specific route can override a broader one without removing it.

Settled 2026-10-08: Qing answered this herself in the [first session](../2026-10-08-first-session.md).

## See also

- [Learning path](../index.md)
