
## Components
**Smithproxy** is heavily multithreaded application. There is single process, launching in certain configurations hundreds of threads. Some for TLS, some for plaintext, some for SOCKS5 handlers.

**Smithproxy** supports also tenants. This is another, advanced topic we won't cover now in detail. You just need to know that tenants are named and indexed.  Default tenant is called "**default**" and its index is **0**.
In `top` tool, you will typically see `sxy_own_0`. This is main thread of default (zero) tenant.
That's how you can correctly locate logs, i.e. `/var/log/smithproxy.0.sslkeylog.log` is SSL keying material dump for default tenant.
