# Scenarios

Let's define some common terms here. If we speak about *observed* or *your host*, we mean the system which will initiate traffic and will get intercepted.
It doesn't assume anything special where it runs, which OS or which level or kind of virtualization.
If we talk about *smithproxy host* we mean the OS where **smithproxy** daemons run -- again, without any prior assumption about virtualization type of level.

## 1. Inline (routed via smithproxy)

**Your goal**:   
You want to go though **smithproxy** with possibly all traffic, but without running anything additionally on *observed host*.
This must be inline operation, where you just change routing information to route packets out from *observed host* to **smithproxy** host.  

Simply drawn it should look like this:
```
                                  
+-- Your host ---+                +-- Smithproxy host ---+
|                |                |                      |
|   [your app]------> OUT ---->   |---  [smithproxy]  -------> OUT
|                |                |                      |
+----------------+                +----------------------+
```

Smithproxy runs on separate VM or host. It's up to you how you design routing. VM can be on the same hardware or you can even leverage the (suboptimal) 
option to use nested virtualization. There is lot of room for imagination and all should work, as long as the routing is correctly set.  

When traffic gets routed on **smithproxy** system, it will get diverted to *TPROXY* ports in **smithproxy**. These start at `50000`, and are specifically 
designed to accept connections *from TPROXY*. 

### Detailed schematics

```

                                  +-- Smithproxy host ------+
                                  |                         |
                                  |  [smithproxy]
                                  |   :udp/50080  plaintext ----> *:*   (same src ip:port) [1]
                                  |   :tcp/50443  tls       ----> *:*   (same src ip:port)
                                  |   :tcp/50080  plaintext ----> *:*   (same src ip:port)
                                      ^      
+-- Your host ---+                    |       
|                |                |   |                     |     
|   [your app]------> OUT ---->   |---  <iptables MANGLE>   |
|                |                |                         |
+----------------+                +-------------------------+

Notes
[1] - Transparency could be enabled/disabled in the policy. 
       Enabled, you need to set up routing to the inside network for returning traffic!
```

This design allows you to route traffic transparently, without setting anything on *observed host* (except trusted CA certificate). Also, connection, leaving **smithproxy** to its upstream router can look pretty same as the original one on *observed host*: source IP and port could be retained. 


> For simplicity of initial setup, **transparency is disabled by default**. You can enable it in the policy once you fix the routing.

### Tips
Best is to run **smithproxy** on separate hardware, or in the VM. With VMs you lose some of speed, notably 
I/O. All my testing VMs in KVM run smoothly. Separate hardware is not typically needed, unless you want to deploy **smithproxy** as experimental firewall or inspect traffic of some specific host on the network which can't be put in the VM.  
If you plan to deploy **smithproxy** in some sort of appliance, please check hardware crypto support of your platform. It's a Good Idea(tm) to have crypto operations supported by CPU for the application (**smithproxy**) which actually perform lot of them.  

## Inline docker scenario

You can run this scenario with both, *smithproxy* **and** *your app* as a docker. Your App container is 
routed via `docker0` interface, which can be, with a bit of setting, also intercepted by smithproxy.

To divert traffic from `docker0` interface, you have to run this hacky thing on the host system :    
`echo "0" | sudo tee /proc/sys/net/bridge/bridge-nf-call-iptables`  

It will enable bridge traffic to end up correctly in TPROXY target (it won't work otherwise). Somebody more knowledgeable
 about docker or iptables can really evaluate risk of this, or better alternative. For me, it's a hack which did the
  job.  

If you want to run smithproxy also in the docker container, run it with `--network host` option, and use
`sx_network`-like diverts from host system to dockerized smithproxy tproxy ports.


### Details 
```

                                  +-- Smithproxy container--+
                                  |      --network host     |
                                  |                         |
                                  |  [smithproxy]
                                  |   :udp/50080  plaintext ----> *:*   (same src ip:port) [1]
                                  |   :tcp/50443  tls       ----> *:*   (same src ip:port)
                                  |   :tcp/50080  plaintext ----> *:*   (same src ip:port)
                                  |    ^      
                                  +--- | -------------------+
                                      
                                       |
+- Your container+                +--  | - Host system ------+     
|                |                |    |                     |     
|   [your app]------> OUT ---->   |--- '  <iptables MANGLE>  |
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


## 2. On the stick (on the same box)

**Your goal**:  
You want to quickly run **smithproxy** and do the thing. You prefer straightforward installation and setup, even at the cost of some functional limitations.

Simply drawn you want something like follows:
```txt
+-- SX host -----+  
|                |  
|   [smithproxy] |  
|                |  
|   [your app]---------------> OUT    
|                |  
+----------------+  
```
Observed host and smithproxy host are the same, maybe your PC or laptop. You cannot use *TPROXY* target here, because locally originated traffic in OUTPUT chain cannot be redirected there. Instead of it, we can use *REDIRECT* target. This target traffic processing is however tricky to implement in **smithproxy** with transparency support for UDP.  
Because vast majority of deployments will not really focus on UDP, we made it partially supported for DNS traffic only. 

### Detailed schematics

```txt
+-- Your AND sx host---------------+  
|
|   [smithproxy]   
|        :tcp/51443 for TLS       ------> *:*               (src ip:port changed)
|        :tcp/51080 for plaintext ------> *:*               (src ip:port changed)
|        :udp/51053 for udp/53    ------> configired_ip:53  (src ip:port changed)
|        ^------------.           
|                     |            |
|   [your app] --->   <iptables OUTPUT> chain as 'user'     
|                 
+----------------------------------+  
```

As you see, *REDIRECT* target smithproxy ports start at `51000`. Specific handling of dns udp traffic is made obvious by changing the port which ends with `53` (`51053`).

### Tips
Best to run this scenario is to install **smithproxy** in the docker. You can redirect traffic from *your host* to the container with simple iptables script. See installation chapter, where it's covered in more detail.

&nbsp;

---


## 3. SOCKS proxy
**Your goal**: Install smithproxy somewhere in the network, while you don't plan to route traffic through it. You want to explicitly use smithproxy if you configure SOCKS aware application to do so (browsers support SOCKS). 

### Detailed schematics
Let's skip simplified picture - it looks like this:

```txt
+-- Your host ----+    --> directly OUT 
|                 |   /
|   [some app]-------'                    +---Smithproxy host -+
|    ...                                  |                    |
|   [observed app]  -----   socks5        |     [smithproxy]
|                 |       \-------------------> :tcp/1080 ---------> tcp *.*   
+-----------------+                                                 (src ip:port of sx host)
                                          +--------------------+
```
Smithproxy accepts proxy connection and handles the request similar way as it were transparent. The trick is that application opens `tcp/1080` connection to **smithproxy** and in the very lightweight handshake it will tell where it want to be connected to. This way **smithproxy** knows where to connect.  
The connection target could be in form of FQDN, so if you see in your application some setting like "send DNS via SOCKS", you can use it.


### Tips
This might work only of your application supports configuring SOCKS5. SOCKS4 should work too. For applications which don't support SOCKS directly,
you can try to *socksify* them.

#### Linux
 In linux, we actually have `socksify`, you can probably install it from package manager.

#### Windows
In Windows, you need something similar to **InjectSOCKS**. We have never tested that one or anything similar, but it looks promising.  Good luck!
