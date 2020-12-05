# Capturing traffic with Smithproxy

Smithproxy is capable of capturing traffic which is passed through it, including intercepted and mitm-ed TLS traffic
. Capturing is controlled by *policy*, using *content_profile* enabling it.

## Capturing policy and profile

See a sample policy:

```
smithproxy(razor) # show config policy 1
policy = ( 
  {
    proto = "tcp"
    src = [ "any", "any6" ]
    sport = [ "all" ]
    dst = [ "any", "any6" ]
    dport = [ "all" ]
    action = "accept"
    nat = "auto"
    tls_profile = "default"
    detection_profile = "detect"
    content_profile = "default"             // <------- this policy is using "default" content profile
    auth_profile = "resolve"
    alg_dns_profile = "dns_default"
  } )
```

Let's check *default* content profile:


```content_profiles = 
{
  default = 
  {
    write_payload = false                   // <------- write_payload controls if the traffic is dumped to files
  }
  // ... 
```

As we see above, *default* profile is not set to write connections content into files. We can change it using cli:


```
smithproxy(razor) # configure terminal 

smithproxy(razor) (config)# edit content_profiles 

smithproxy(razor) (config-content_profiles)# edit 
  default             edit default settings
  evil                edit evil settings

smithproxy(razor) (config-content_profiles)# edit default 

smithproxy(razor) (config-default)# set write_payload true
running config applied (not saved to file).
change applied to current config

smithproxy(razor)<*> (config-default)# exit

smithproxy(razor)<*> (config)# exit

```

## Capture files and their content

Let's check now if files are being written into the files. Note also how they are stored and organized into directories.

```root@razor:/var/lib/docker/volumes/sxydumps/_data# find .
   .
   ./data
   ./data/172.30.1.102
   ./data/172.30.1.102/2020-12-05
   ./data/172.30.1.102/2020-12-05/14-14-14_ssli_172.30.1.102:34708-ssli_40.127.110.237:443.smcap
   ./data/172.30.1.102/2020-12-05/14-14-14_ssli_172.30.1.102:35958-ssli_68.232.34.200:443.smcap
   ./data/172.30.1.102/2020-12-05/14-14-43_ssli_172.30.1.102:38846-ssli_52.114.133.168:443.smcap
   ./data/127.0.0.1
   ./data/127.0.0.1/2020-12-05
   ./data/127.0.0.1/2020-12-05/14-14-43_udp_127.0.0.1:53230-udp_8.8.4.4:53.smcap
   ./data/127.0.0.1/2020-12-05/14-14-14_udp_127.0.0.1:39975-udp_8.8.4.4:53.smcap
   ./data/127.0.0.1/2020-12-05/14-14-14_udp_127.0.0.1:33418-udp_8.8.4.4:53.smcap
   root@razor:/var/lib/docker/volumes/sxydumps/_data# 
```

We can have a look inside, ie. this one:

```
root@razor:/var/lib/docker/volumes/sxydumps/_data# cat ./data/127.0.0.1/2020-12-05/14-15-26_udp_127.0.0.1:33829-udp_8.8.4.4:53.smcap
Sat Dec  5 14:15:26 2020
+111179: udp_127.0.0.1:33829-udp_8.8.4.4:53(udp_8.8.4.4:53-udp_127.0.0.1:33829)
Connection start

Sat Dec  5 14:15:26 2020
+111986: udp_127.0.0.1:33829-udp_8.8.4.4:53(udp_8.8.4.4:53-udp_127.0.0.1:33829)
>[0000]   40 63 01 20 00 01 00 00   00 00 00 01 03 77 77 77   @c...... .....www
>[0010]   04 61 6C 7A 61 02 63 7A   00 00 01 00 01 00 00 29   .alza.cz .......)
>[0020]   04 B0 00 00 00 00 00 00                             ........ 

Sat Dec  5 14:15:26 2020
+112135: udp_127.0.0.1:33829-udp_8.8.4.4:53(udp_8.8.4.4:53-udp_127.0.0.1:33829)
>[0000]   5C 5E 01 20 00 01 00 00   00 00 00 01 03 77 77 77   .^...... .....www
>[0010]   04 61 6C 7A 61 02 63 7A   00 00 1C 00 01 00 00 29   .alza.cz .......)
>[0020]   04 B0 00 00 00 00 00 00                             ........ 

Sat Dec  5 14:15:26 2020
+150435: udp_8.8.4.4:53-udp_127.0.0.1:33829(udp_127.0.0.1:33829-udp_8.8.4.4:53)
     <[0000]   5C 5E 81 A0 00 01 00 00   00 01 00 01 03 77 77 77   .^...... .....www
     <[0010]   04 61 6C 7A 61 02 63 7A   00 00 1C 00 01 C0 10 00   .alza.cz ........
     <[0020]   06 00 01 00 00 04 46 00   34 04 61 6C 66 61 02 6E   ......F. 4.alfa.n
     <[0030]   73 08 61 63 74 69 76 65   32 34 C0 15 0A 68 6F 73   s.active 24...hos
     <[0040]   74 6D 61 73 74 65 72 C0   31 78 68 77 AD 00 00 2A   tmaster. 1xhw...*
     <[0050]   30 00 00 07 08 00 12 75   00 00 00 0E 10 00 00 29   0......u .......)
     <[0060]   02 00 00 00 00 00 00 00                             ........ 

Sat Dec  5 14:15:26 2020
+150580: udp_8.8.4.4:53-udp_127.0.0.1:33829(udp_127.0.0.1:33829-udp_8.8.4.4:53)
     <[0000]   40 63 81 A0 00 01 00 01   00 00 00 01 03 77 77 77   @c...... .....www
     <[0010]   04 61 6C 7A 61 02 63 7A   00 00 01 00 01 C0 0C 00   .alza.cz ........
     <[0020]   01 00 01 00 00 00 FA 00   04 B9 B5 B0 13 00 00 29   ........ .......)
     <[0030]   02 00 00 00 00 00 00 00                             ........ 

Sat Dec  5 14:15:39 2020
+62212: udp_127.0.0.1:33829-udp_8.8.4.4:53(udp_8.8.4.4:53-udp_127.0.0.1:33829)
Connection stop


```

The above is capture of DNS traffic querying alza.cz domain. Captures look similar also for other traffic, regardless
 it's TCP, TLS, or something else supported (and intercepted).
 
 
## Capture disabled flag file

Sometimes one doesn't want to go to CLI or reload smithproxy to stop normally enabled traffic capture. If you create
 file named `disabled` in the capture root directory, smithproxy will not save the capture, even though it's
  configured in the matching content profile.
 
 File is for example on this place (depends how is set your capture root dir): 
 
 ```
root@razor:/var/lib/docker/volumes/sxydumps/_data/data# touch disabled
root@razor:/var/lib/docker/volumes/sxydumps/_data/data# ls -lsa
total 16
4 d-w-r-xr-T 4 root root 4096 pro  5 15:26 .
4 drwxr-xr-x 3 root root 4096 pro  5 15:26 ..
4 drwxr-x--- 3 root root 4096 pro  5 15:14 127.0.0.1
4 drwxr-x--- 3 root root 4096 pro  5 15:14 172.30.1.102
0 -rw-r--r-- 1 root root    0 pro  5 15:26 disabled                # <------ disables traffic captures
```
 
> Note: there is one drawback - the content is still prepared, the saving procedure is just skipped, implying all CPU
> costs to prepare the content to be written are wasted. Converting binary into formatted hex dump is quite expensive
>, so use this technique only as temporary solution. 
  
  
## Uses of smithproxy *.smcap files

Sole hex dump is useful already - you can just have a look into any, originally encrypted traffic. There is however
 also the possibility to **replay the traffic** using `pplay` tool, which is smithproxy sister project.
 
We are aware that naming is a bit confusing with other tool for replay videos. Our `pplay` will be renamed to something
 more fitting.
    
 
 > ** Have a look at [pplay homepage](https://pypi.org/project/pplay/). It's far beyond to be just smcap replayer.It
> supports pcap files and has a lot of interesting features. 
