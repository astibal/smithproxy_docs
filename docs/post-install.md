
# Post-install instructions

This chapter should help you with next steps, after you sucessfully installed **smithproxy**.

## Quick info

- **Certificate Authority**: First thing to change: CA certificates!  
  Smithproxy is performing MitM attack on passing TLS connections, so your clients must trust it!
  
- **Redirection**: Check interface setup and redirection options (usually it just works)  
  file `/etc/smithproxy/smithproxy.startup.cfg` contains networking setup options to check and modify
  
> You can check [quick start from snap](https://www.youtube.com/watch?v=_uhKHmmKFL8) yt video covering basic installation and how to configure certificates. Video is shows the snap installation, but the steps are generally valid.


## Certificate Authority

### How it works (you can skip it)

First, let's briefly recap what SSL inspection works. There is a middlebox. This is in our case **Smithproxy**. This middlebox function is to accept TLS connection, pretending it's real target server. There is no magic way to break cryptography in TLS of any passing connection (fortunately).

Therefore, middlebox must accept TLS connection with different certificate and its own (possibly very similar), cryptographical properties as real server. To do so, middlebox, just before accepting TLS from client, connects to real server, and copies the certificate details, signing it with its own, by middlebox maintained certificate authority. This middlebox CA must be of course trusted by client, otherwise connection won't be trusted and ultimately fails.

> Note: before connecting to real server, middlebox must know SNI (or it's absence), before connecting to real server. Otherwise connections to server with multiple TLS virtual servers would fail, or fall to default virtual host. That being said, knowing target IP address is not enough.

For more information, please read more [Details about Smithproxy CA](../ca-ops).


## Understanding redirection

To intercept traffic, you have to somwhow get it to smithproxy first. This is done using iptables in pre-made scripts
. There are few options in this regard: 

* Redirect local traffic to locally running smithproxy (using iptables REDIRECT target)
* Intercept forwarded (routed) traffic through the host running smithproxy (using iptables TPROXY traffic)
* Process proxied traffic from remote browser configured for SOCKS proxy running on host running smithproxy (no
 iptables redirection needed)

You can combine all above in single smithproxy process. However, for each of targets above, separate thread tree is
 spawned consuming its own IO and CPU. Specifically idle CPU can be few percents when all targets are running.  
 
 > Use only features you really need. Target features can be disabled separately in the configuration using
> `settings/accept_*` directive.
