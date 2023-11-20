# Scenarios

## Couple of terms
- "routed traffic" - network traffic, where both source and destination addresses don't belong to this host  
- "locally originated traffic" - network traffic, which has been initiated by this host
- "to divert traffic" - to make "routed traffic" recognized as "locally originated" (socket with non-local address) 

- `TPROXY` - iptables target where routed traffic can be diverted to (`mangle` table)
- `REDIRECT` - iptables target where locally originating traffic is diverted to (`nat` table)


Consider we have:  
- **app host** - host with the application initiating traffic we are interested to   
- **smithproxy host** we mean the OS where **smithproxy** is running 

## 1. Inline (routed via smithproxy) ** PREFERRED **

**Your goal**:   
You want to pass all your traffic through **smithproxy**, but without running anything additionally on *app host*.
This must be inline operation, where you just change routing information to route packets out from *app host* to **smithproxy** host.  

```
                                  
+--- App host ---+              +-- Smithproxy host ---+
|                |              |                      |
|   [your app]------> OUT --->  |---  [smithproxy]  -------> OUT
|                |              |                      |
+----------------+              +----------------------+
```

Smithproxy can run on a separate VM or host, requiring only basic routing setup.
This setup is both highly flexible and well-tested. As long as the routing is correctly set, you should be good.  

When traffic gets routed on **smithproxy** system, it will get diverted to *TPROXY* ports in **smithproxy**. 

Detail:
```

                                  +-- Smithproxy host ------+
                                  |                         |
                                  |  [smithproxy]
                                  |   :udp/50080  plaintext ----> *:*   (same src ip:port) 
                                  |   :tcp/50443  tls       ----> *:*   (same src ip:port)
                                  |   :tcp/50080  plaintext ----> *:*   (same src ip:port)
                                  |   ^                                 [1]
+--  App host ---+                |   |       
|                |                |   |                     |     
|     [ app ] ------> OUT ---->   |---  <iptables MANGLE>   |
|                |                |                         |
+----------------+                +-------------------------+

Notes
[1] - The transparency feature can be enabled or disabled within the policy settings. 
      If enabled, you need to set up routing to the inside network for returning traffic!
```

This design allows you to route traffic transparently, without setting anything on *app host* (except trusted CA certificate). 
Also, connection, leaving **smithproxy** to its uplink router can look the same as the original one on *app host*. 

> To simplify the initial setup process, transparency is turned off by default.

---

## 2. On the stick (on the same box)

**Your goal**:  
If you're aiming for a quick setup and start with **smithproxy** right away. You want quick installation and setup,  
even at the cost of some functional limitations.

```txt
+-- SX host -----+  
|                |  
|   [smithproxy] |  
|                |  
|   [your app]---------------> OUT    
|                |  
+----------------+  
```

Observed host and smithproxy host are the same, maybe your PC or laptop. 
> Due to technical limitations, only UDP traffic for DNS is supported.


Detailed:
```txt
+--- App AND sx host---------------+  
|
|   [smithproxy]   
|        :tcp/51443 for TLS       ------> *:*               (src ip:port changed)
|        :tcp/51080 for plaintext ------> *:*               (src ip:port changed)
|        :udp/51053 for udp/53    ------> configired_ip:53  (src ip:port changed)
|        ^------------.                        
|                     |            
|                     |
|   [ app] --->   <iptables OUTPUT> chain as 'user'     
|                 
+----------------------------------+  
```

For locally initiated traffic we must use `REDIRECT` target in `nat` chain.  
Because of this, we won't see original target IP address for UDP. UDP works only for DNS traffic,  
which is hardwired to be resent to smithproxy configured DNS server.


---

## 3. SOCKS proxy

**Your goal**:   
You want or already have smithproxy somewhere in the network. You don't plan to route traffic through it.  
All is just about to configure SOCKS5 in the app.

Detailed:
```txt
+-- App  host ----+    --> directly OUT 
|                 |   /
|   [some app]-------'                    +---Smithproxy host -+
|    ...                                  |                    |
|   [observed app]  -----   socks5        |     [smithproxy]
|                 |       \-------------------> :tcp/1080 ---------> tcp *.*   
+-----------------+                                                 (src ip:port of sx host)
                                          +--------------------+
```

Smithproxy accepts SOCKS5 proxy connections and handles the request similar way as it were transparent,
including FQDN names (often referred as "proxy DNS when using SOCKS v5").
  
Starting with smithproxy `0.9.30`, you can also use UDP SOCKS5, but it's quite rare requirement 
and you most likely won't ever need it.

#### Linux
Browsers support SOCKS5 and so do also many apps too.   
The rest can be made by using `socksify` tools (i.e. `dante`).

#### Windows
In Windows, you need something similar to **InjectSOCKS** or **Proxifier**.   
I have never tested any, but it looks promising.  Good luck!

--- 

## Inline docker scenario

**Your goal**:   
You like your life too complicated ... or found yourself in situation you have to intercept docker containers.

Fortunately, you can run both **smithproxy** and **your application** within separate Docker containers. 
Your App container is routed via `docker0` interface intercepted by smithproxy.

To divert traffic from `docker0` interface, you have to run this hacky thing on the host system :    
`echo "0" | sudo tee /proc/sys/net/bridge/bridge-nf-call-iptables`

It will enable bridge traffic to end up correctly in TPROXY target (it won't work otherwise). Somebody more knowledgeable
about docker or iptables can really evaluate risk of this, or better alternative. For me, it's a hack which did the
job.

If you want to run smithproxy also in the docker container, run it with `--network host` option, and use
`sx_network`-like diverts from host system to dockerized smithproxy tproxy ports.

Details:
```

                                  +-- Smithproxy container--+
                                  |      --network host     |
                                  |                         |
                                  |  [smithproxy]
                                  |   :udp/50080  plaintext ----> *:*   (same src ip:port) 
                                  |   :tcp/50443  tls       ----> *:*   (same src ip:port)
                                  |   :tcp/50080  plaintext ----> *:*   (same src ip:port)
                                  |    ^                                 [1] 
                                  +--- | -------------------+
                                      
                                       |
+- App container-+                +--  | - Host system ------+     
|                |                |    |                     |     
|   [ app ]------> OUT ---->   |--- '  <iptables MANGLE>  |
|                |                |       on docker0 intf    |
+----------------+                +--------------------------+

Notes
[1] - Transparency could be enabled/disabled in the policy. 
       Enabled, you need to set up routing to the inside network for returning traffic!
```

Host system MANGLE rules can be inspired by these in `sx_network` script. Also, don't forget `docker0` interface
will appear in the system only after a container is running. Mangle scripts should be therefore run after `docker0
` interfaces are created.

> `/etc/smithproxy/smithproxy.startup.cfg`: magic `'-'` interface setting `SMITH_INTERFACE='-'` will ensure all downlink
> (not having default route associated) interfaces are intercepted.



&nbsp;

---
