# Shell command reference

> Note: this is not CLI command reference

> Note: snap installations are simplified for quick use and don't have all commands below installed

## `sx_ctl` command

Usage:
```
sx_ctl start|stop|status|fix
```
This command will control full smithproxy instance, including:
 * smithproxy core
 * smithd
 * bend
 * web portal for HTTP
 * web portal for HTTPS

## `smithproxy` command
Usage:
```
smithproxy [--tenant-id <name> --tenant-idx <num>] [--daemonize] [--help] [--]
```
Parameters (mandatory):  
 * *(no mandatory parameters)*

Parameters (optional):
 * `tenant-id` name of tenant, it is a base string for logs
 * `tenant-id` tenant index, number for lookup in tenant table
 * `daemonize` run smithproxy at background
 * `--config-check-only` load config file to check its correctness

Program is core smithproxy component. Run without arguments, it loads `smithproxy.cfg` configuration 
file and starts to run on foreground unless specified otherwise.


## `sx_regencerts` command
Usage:
```
sx_regencerts
```
Signing CA authority generator script

Program doesn't take any arguments. It will check smithproxy certificate store and interactively
 asks user for desired actions. it can generate new signing CA and necessary keys for EC or RSA methods.   
 Program can be executed in *dry* mode, to just check how it works.


## `sx_cli` command
```
sx_cli [tenant_nane]
```
This is simple script connecting administrator to smithproxy CLI interface. It is tenant aware.  
If tenant table is present in the system, tenant_name is mandatory.

## `sx_network` command

Script to prepare networking in the system for smithproxy use. 
Usage:

```
sx_network start|stop [tenantid]
```
If smithproxy is configured for multiple tenants, you need to specify tenant name as the last 
argument (it is not needed otherwise).

Networking setup is configurable in the `/etc/smithproxy/smithproxy.startup.cfg` file. 
It controls which traffic is diverted to smithproxy and which interfaces it is related to.

There are many options, but let's highlight some:  

* `SMITH_INTERFACE='-'`
 Special value `-` - apply divert rules to all interfaces without default route
 Special value `*` - apply divert rules to all interfaces in the system
 Any other string value means interface name.

* `SMITH_TCP_PORTS_ALL=1`

Inspect all TCP traffic

## `sx_passwd` command

This command checks or changes user password. Users are stored in separate 
`/etc/smithproxy/users.cfg` file. User secrets are salted and encrypted by 
`/etc/smithproxy/users.key` (128 byte key). 
Obviously, if you change it, all secrets are lost.  

If you want use smithproxy with authentication in serious way, you will probably want
to generate new key file and not use default pre-installed one.  

If used with `--check` argument, sx_passwd process will return 0 if password check was ok, 
or 1 otherwise. 

Usage examples:
```
# change password for user 'abc' (with password prompt)
sx_passwd --user abc             

# change password for user 'abc' to MySecretPass
sx_passwd --user abc --password MySecretPass 

# check user 'abc' password
sx_passwd 
```

## `sx_download_ctlog` command

To technically allow outbound TLS connections certificate transparency checks (which are enabled by default), 
you need to download CT keys log.    
`sx_download_ctlog` will download ready-made log prepared in smithproxy.org download site.
Downloaded CT log is stored in `/etc/smithproxy/ct_log_list.cnf` file. 


## `smithd` command

This deamon is query server ready for future use. It's not an active component currently.
