# Welcome to smithproxy documentation

## Project overview

> Smithproxy is fast and featured TPROXY-based TCP/UDP/TLS transparent proxy for linux.

It's highly configurable with traffic policies and profiles. Smithproxy does have its own CLI where you can
configure it and observe various diagnostics and other outputs.

It should run on any recent linux system, but it requires `root` privileges if you want it to be fully transparent and not changing source IP and port of passing connections. You can connect to smithproxy also using SOCKS5 protocol.

Smithproxy parially understands DNS traffic, you can use FQDN or domain wildcards for making policy match.
Also, together with HTTP, you can redirect browser traffic to login captive portal to authenticate user.
Captive portal redirection is correctly signed by trusted CA, if configured properly.

## When and where to use it

Smithproxy is kind of dual-purposed, or if you prefer, double-faced.

### Attack proxy
You can use smithproxy for (*simulating*) malicious activities. It can capture plaintext traffic or SSL keys. Traffic can be later analyzed with other tools, or replayed. Traffic can be also replaced with a custom payload.

### Experimental firewall
Smithproxy performs by default all possible TLS checks. STARTTLS support, OCSP for all connections, hash strenght checks, etc. Its policy granularity provides quite good controls over the traffic.  



## Components
Smithproxy is heavily multithreaded application. There is single process, launching in certain configurations hundreds of threads. Some for TLS, some for plaintext, some for SOCKS5 handlers.

Smithproxy supports also tenants. This is another, advanced topic we won't cover now in detail. You just need to know that tenants are named and indexed.  Default tenant is called "**default**" and its index is **0**.
In `top` tool, you will typically see `sxy_own_0`. This is main thread of default (zero) tenant.
That's how you can correctly locate logs, i.e. `/var/log/smithproxy.0.sslkeylog.log` is SSL keying material dump for default tenant.
