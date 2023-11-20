# Installation - docker

## Super easy to test 

For a test, just simply run: 
> `sudo docker run -it --shm-size 512M --rm astibal/smithproxy:latest`  

>  Note: with default `shm-size` 64MB smithproxy will terminate, it needs `256M` as minimum 

This will download latest **smithproxy** build and run it. You can see CLI, and other features.
However, this is usually not what you really want long term - there are options you want to keep and persist.

## Docker tags

Above example pulls `latest` tag. Which is build from `master` branch in the Ubuntu 18.04 LTS.

Docker tags:

* `latest` - Ubuntu 18.04 LTS + master branch (fresh, it-should-work option)


## Volumes

There are at least three places you need to keep to restart docker without losing your data:

- `/etc/smithproxy/` - dir, all config is placed here
- `/var/log/smithproxy/` - keep your logs (including SSL key dumps)
- `/var/local/smithproxy/` - captures default storage

## Shared memory

Smithproxy will **not** run properly with less than ~200MB of shared memory and it will terminate. 
Startup scripts are adjusted to allocate 512MB of shm, you can decrease this number a bit if needed. 
But never ho below minimal amount.  

Shared memory is used to synchronize identity data from other smithproxy components. If shm is depleted,
program is terminated with SIGBUS. This happens few tens of seconds after the start when tables are id refreshed.  

## Docker launcher script
This script creates volumes *if needed* and attaches them to
correct places in the container. Run this in the *host* system.

```bash
#!/bin/bash

TAG="latest"

TMPFS="no"
LOG_VOLUME="--mount type=tmpfs,destination=/var/log,tmpfs-size=1000000000"

if [ "$1" != "" ]; then
    TAG=$1
    shift
fi

( sudo docker inspect sxy ) > /dev/null 2>&1; 
SXY_=$?
( sudo docker inspect sxyvar ) > /dev/null 2>&1; 
SXYVAR_=$?
( sudo docker inspect sxydumps ) > /dev/null 2>&1; 
SXYDUMPS_=$?

if [ "$SXY_" != "0" ]; then
    echo "... creating /etc volume"
    sudo docker volume create sxy
fi

if [ "$TMPFS" != "yes" ]; then
    if [ "$SXYVAR_" != "0" ]; then
        echo "... creating /var/log volume"
        sudo docker volume create sxyvar
    fi
    LOG_VOLUME="-v sxyvar:/var/log"
fi
    
if [ "$SXYDUMPS_" != "0" ]; then
    echo "... creating /var/local/smithproxy volume"
    sudo docker volume create sxydumps
fi

sudo docker pull astibal/smithproxy:${TAG}
sudo docker run -v sxy:/etc/smithproxy \
        ${LOG_VOLUME} \
        -v sxydumps:/var/local/smithproxy \
        -it \
        --shm-size 512M \ 
        --rm --network host --name "sx-${TAG}" astibal/smithproxy:${TAG} "$@"

```

**NOTE**: If you are not interested in logs and don't mind to lose some of memory to save disk space and some IO, set `TMPFS="yes"`. 
It will mount /var/log as a ramdisk. You can easily adapt this script to do something similar to captures, if you want.


## Post-install specific to docker

### SOCKS5
SOCKS5 is ready to use right out of the box. Just configure you application to use it.

### Locally originated traffic
As a docker user, you probably want to redirect local traffic from host (your PC) to smithproxy in docker container.  
This is however a bit tricky, because you want to divert only user traffic, not traffic already passed through smithproxy.

Here comes another script helping with that, too (run this in the host system):

```bash
#!/usr/bin/env bash

EXEMPT_USERS="root fahclient boinc"

case "$1" in

start)
    iptables -t nat -F OUTPUT
    # comment out if you want to redirect also for LAN traffic
    iptables -t nat -A OUTPUT -p tcp -d 10.0.0.0/8 -j ACCEPT
    iptables -t nat -A OUTPUT -p tcp -d 172.16.0.0/12 -j ACCEPT
    iptables -t nat -A OUTPUT -p tcp -d 192.168.0.0/16 -j ACCEPT
    # --
    iptables -t nat -A OUTPUT -p tcp -d 127.0.0.0/8 -j ACCEPT
    iptables -t nat -A OUTPUT -p tcp -s 127.0.0.0/8 -j ACCEPT

    for U in $EXEMPT_USERS; do
        iptables -t nat -A OUTPUT -m owner --uid-owner ${U} -j ACCEPT
    done

    iptables -t nat -A OUTPUT -p tcp --dport 443 -j REDIRECT --to-port 51443  \
             -m owner ! --uid-owner root
    iptables -t nat -A OUTPUT -p udp --dport 53 -j REDIRECT --to-port 51053   \
             -m owner ! --uid-owner root
    iptables -t nat -A OUTPUT -p tcp -j REDIRECT --to-port 51080  \
             -m owner ! --uid-owner root
    ;;

stop)
     iptables -t nat -F OUTPUT
    ;;

esac
```
This will exempt local LAN traffic and some users' traffic from smithproxy redirection. Because smithproxy runs as a root,
it's exempted too, and not redirected back to itself.

**NOTE**: Bear in mind, generic UDP is not supported with REDIRECT iptables setup.
This is why we redirect `udp/53` to port `udp/51053`, which assume it's DNS and forwards it to DNS servers set in `smithproxy.cfg`.
Other traffic is handled correctly.  

** Also, don't forget to follow post-installation instructions common for all smithproxy scenarios. This chapter covers only docker specifics.**


## Supported features

|&nbsp;Feature&nbsp;|&nbsp;Support&nbsp;|&nbsp;&nbsp;Method&nbsp;&nbsp;|&nbsp;Notes&nbsp;|
|:---:|:---:|:---:|:---|
|- **routed** traffic -| &nbsp;- not tested -&nbsp; |&nbsp;tproxy  | it may work normally using TPROXY     |
|- **local** traffic - | &nbsp;- supported -&nbsp;  |&nbsp;redirect| Limitation: UDP only for DNS on 51053 |
|**socks**             | &nbsp;- supported -&nbsp;  |        |                                       |


&nbsp;
