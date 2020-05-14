# Welcome

## Overview

> **Smithproxy** is fast and featured tproxy-based TCP/UDP/TLS transparent proxy for linux.

It's highly configurable with objects, traffic policies and profiles. **Smithproxy** also has its CLI where you can
configure it and observe various diagnostics and other outputs.

**Smithproxy** should run on any recent linux system, but it requires `root` privileges if you want it to be fully transparent and not changing source IP and port of passing connections. You can connect to smithproxy also using SOCKS5 protocol.

**Smithproxy** parially understands DNS traffic, you can use FQDN or domain wildcards for making policy match.
Also, together with HTTP, you can redirect browser traffic to login captive portal to authenticate user.
Captive portal redirection is correctly signed by trusted CA, if configured properly.

**Smithproxy** is considerably fast. If you have alredy tried other (ie. python-based) TLS mitm proxies, you should see significant speed-up! Browsing complex http pages should not be very different from going there directly.

## Typical Usage

**Smithproxy** is kind of dual-purposed, or if you prefer, double-faced.

### Attack proxy
You can use smithproxy for (*simulating*) malicious activities. It can capture plaintext traffic or SSL keys. Traffic can be later analyzed with other tools, or replayed with [pplay](https://pypi.org/project/pplay/) which directly supports smithproxy `*.smcap` captures (and also `*.pcap[ng]`). Traffic can be also replaced with a custom payload.

### Experimental firewall
**Smithproxy** performs by default all possible TLS checks. STARTTLS support, OCSP for all connections, hash strenght checks, etc. Its policy granularity provides quite good controls over the traffic.  

