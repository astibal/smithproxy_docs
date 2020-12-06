# Installation from sources

This is actually very easy. Only drawback is, you (script) will will need to install some development 
tools in order to build and make work everything correctly.

You will need to initially install git, everything else is downloaded automatically later by script.

## Supported platforms

- **amd64** - tested on various VMs and barebone machines
- **aarch64** - tested on [RockPro64 board](https://www.pine64.org/rockpro64/)  
  Note: many ARM cpus don't have [crypto extension](https://en.wikichip.org/wiki/arm/armv8#Crypto_Extension), which come really handy in **smithproxy**!

## Distribution support

**Directly supported**:

- Ubuntu >= 18.04 (and derivatives including Kali Linux)
- Fedora >= 31
- Debian >= 10
- Alpine >= 3.11

Above distros have specific support in the OS detection script, which will install dependencies as needed. You can just skip to **A/** or **B/**.  
Of course, building from sources works in docker container too. 

### Unsupported distros

- Amazon linux 2  - proven to not work due to ancient/conservative software versions. 

### Directly unsupported - may work
Any other ditribution may, but doesn't have to, work correctly.

If in doubt, first check compiler. It **must** be capable to compile C++17 sources (for GCC it's `>= 8.3`).
To install all dependencies, have a look in `tools/docker/0.9/run` where is `linux-deps.sh`. Find most similar distribution and try to download dependencies yourself.  
Good Luck!

## A. Use installer script

```bash
wget -O - https://www.smithproxy.org/get/installer.sh | sh -
```

## B. Get sources yourself

```bash
git clone https://github.com/astibal/smithproxy.git -b master
cd smithproxy
./linux-deps.sh && linux-build.sh
```
**Note**: you can replace `master` from above snippet with your desired version or tag which is `>= 0.9`. This is also what installer from chapter A/ is doing.  
