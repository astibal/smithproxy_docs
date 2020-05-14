
# Daemons

**Smithproxy** is quite complex and heavily multithreaded application. Startup script has to initialize iptables, load tenants (not enabled by default) and run the watchdog process to launch all necessary executables.
Core smithproxy doing the actual proxy work is launching in certain configurations hundreds of threads. Some for TLS, some for plaintext, some for SOCKS5 handlers.

See now how is the sequence.


## Startup

- `/etc/init.d/smithproxy`
    - **Startup script** 
    -  *optional*: read tenant config from `/etc/smithproxy/smithproxy.tenants.cfg` and run for all below 

    - `/etc/smithproxy/smithproxy.startup.sh` read `/etc/smithproxy/smithproxy.startup.cfg`
        - **Redirection setup**
        - *optional*: you can tune up ports and various redirection aspects in `.cfg`
        - *optional*: create iptables rules to divert traffic 
    
    - `/usr/share/smithproxy/infra/smithdog.py` 
        - **Watchdog daemon**
        - execute and keep alive core components and daemons (if any crashes, it loads it again)
        - Components:
            - `/usr/bin/smithproxy` - core binary, heavily multithreaded (instance per tenant)
            - `/usr/bin/smithd` - query daemon (single instance) *for future use*
            - "bend" - SOAP service backend for integration services (currently authentication)
            - "portals" - http and https portal to login user



## Note on tenants
**Smithproxy** supports also tenants. What they actually are? They are separately run smithproxy instances with some (or all), shared configuration parts.  

Tenants are separated by iptables (by IPv4 and IPv6 address). We will cover it more in next documentation chapters. 
In component point of view, you need to know that tenants daemens are distunguished by the daemon name and suffix.  

Default tenant is called "**default**" and its index is **0**.
In `top` tool, you will typically see `sxy_own_0`. This is main thread of default (zero) tenant.
That's how you can correctly locate logs, i.e. `/var/log/smithproxy.0.sslkeylog.log` is SSL keying material dump for default tenant.
This is also default configuration, if you launch smithproxy for the first time, you will see just one, default tenant to run.
