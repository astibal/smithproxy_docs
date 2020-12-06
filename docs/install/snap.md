# Installation - snap

Snap is Ubuntu-oriented app store. You can install smithproxy from there.

> Note: the functionality is limited for smithproxy snap and usage is slightly different.  
> It's recommended only for local traffic captures, portal services are not supported.

> **Use sudo (or run a root shell) on all smithproxy components** 

```
sudo snap install smithproxy --edge

sudo snap connect smithproxy:firewall-control
sudo snap connect smithproxy:network-control   
```  

Now you have smithproxy installed and connected to all plugs needed to be running properly. 
 * `firewall-control` plug is needed to get iptables diverting traffic correctly
 * `network-control` plug is required to investigate IP routing to select inbound and outbound ports


## Initial smithproxy snap setup 


* Display signing CA information: 
    ```
    sudo smithproxy.certinfo-ca
    ```

* Generate a new signing CA:
    ```
    sudo smithproxy.regencert
    ```

* From previous output copy displayed CA public key and save it somewhere

* Use saved CA public key file and import it to trusted CAs in browser on application you will test


## Starting smithproxy


* Set up traffic redirection
    ```
    sudo smithproxy.net start
    ```

* Run smithproxy itself
    ```
    sudo smithproxy.exe 
    ```

## Accessing CLI

You can access CLI using telnet to `localhost:50000`. Non-snap smithproxy packages come with `sx_cli` command 
doing the same, snap however won't allow to install telnet application into itself. So telnet is not included and 
therefore you have to connect to CLI yourself:

```
telnet localhost 50000
``` 

## Brief intro to snap filesystem locations

Consider this directory structure of `smithproxy` snap:

```
xu@cr4:~$ find /var/snap/smithproxy/ -type d,l -ls | grep -v pem
 524433      4 drwxr-xr-x   5 root     root         4096 Dec  6 14:11 /var/snap/smithproxy/
 548946      4 drwxr-xr-x   4 root     root         4096 Nov 23 17:43 /var/snap/smithproxy/144/etc/smithproxy
 548955      4 drwxr-xr-x   2 root     root         4096 Nov 23 17:43 /var/snap/smithproxy/144/captures
 548953      4 drwxr-xr-x   3 root     root         4096 Nov 23 17:43 /var/snap/smithproxy/144/ca-bundle
 548952     20 drwxr-xr-x   2 root     root        20480 Dec  6 14:11 /var/snap/smithproxy/144/certs
 548956      4 drwxr-xr-x   2 root     root         4096 Nov 23 17:46 /var/snap/smithproxy/144/log
 529916      4 drwxr-xr-x   2 root     root         4096 Nov 23 17:43 /var/snap/smithproxy/common
 524373      0 lrwxrwxrwx   1 root     root            3 Dec  6 14:11 /var/snap/smithproxy/current -> 144
 524361      4 drwxr-xr-x   8 root     root         4096 Nov 23 17:43 /var/snap/smithproxy/129
 529980      4 drwxr-xr-x   4 root     root         4096 Nov 23 17:43 /var/snap/smithproxy/129/etc/smithproxy
 799904      4 drwxr-xr-x   2 root     root         4096 Nov 23 17:43 /var/snap/smithproxy/129/captures
 798688      4 drwxr-xr-x   3 root     root         4096 Nov 23 17:43 /var/snap/smithproxy/129/ca-bundle
 530004     20 drwxr-xr-x   2 root     root        20480 Dec  1 08:38 /var/snap/smithproxy/129/certs
 799905      4 drwxr-xr-x   2 root     root         4096 Nov 23 17:46 /var/snap/smithproxy/129/log
 
```

Numbers `129` and `144` are smithproxy *build* numbers. Whenever you upgrade snap, 
files are copied to a new location. If you are looking for logs and captures, always look
 into  
`/var/snap/smithproxy/current`   
directory. 

Because snap files organization differs a bit compared to common places on regular installs, 
let's check where is what.

Content of `/var/snap/smithproxy/current`:  
 * `ca-bundle`: public root authorities bundle delivered with snap (snap doesn't have access to your `/etc/ssl`)  
 * `ca-certificates`: *TBA*, empty time being  
 * `captures`: contains all capture files you took  
 * `log`: logs :)   
 * `smithproxy.default.sslkeylog.log` SSL key dump - usable in wireshark   



## Check youtube video
There is [youtube video](https://www.youtube.com/watch?v=_uhKHmmKFL8) demonstrating above steps. 
If you read the doc down here, you will better understand what's going on there. 
