# Welcome

> **Smithproxy** is fast and featured TCP/UDP/TLS transparent + socks5 proxy for linux.  
> Primary aim is to provide best-possible file and GRE export capturing experience, while being:
> * fast (written in C++)  
> * highly configurable (powerful CLI and configuration options)  
> * secure (extensive TLS security options, STARTTLS)  
> * flexible ( You can connect to smithproxy also using SOCKS5 protocol.)  

> For a quick overview how to run basic tasks, check our [YouTube channel](https://www.youtube.com/channel/UCb7BciVQp2pdQw9ndueTlvA).  
> You can also come over to say hi, or ask for help on our [Discord server](https://discord.gg/vf4Qwwt)!

## Traffic capture
You can capture traffic to files, or to resend it in GRE as a remote capture. Indeed, both PCAP file and GRE-encapsulated
captures are plaintext, "decrypted" if you will.  

##  CLI
CLI is highly configurable with objects, traffic policies and profiles. CLI also handles configuration,
you don't need to edit (with very few exceptions) configuration files at all.

## Performance
**Smithproxy** will run on any recent linux system, but it requires `root` privileges to exploit all transparency options.
Overall performance will benefit from multiple CPU cores, as the traffic is internally load-balanced to multiple threads.

## Protocol support
**Smithproxy** understands DNS (+DoH), having HTTP1 and HTTP2 engines. You can monitor these in CLI, too.
So-called "policy features" can further enhance traffic inspection. 

## Webhooks
If configured, **smithproxy** will notify webhook server with connection-related information.  
Note: introduced in 0.9.32.

## Usage variability
**Smithproxy** can be well a hacking/capturing proxy, or your experimental network firewall.   
With the newest _webhook_ and _policy features_ additions, it can be used as a honeynet network firewall. 

