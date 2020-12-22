# Shell command reference

> Note: this is not CLI command reference

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

Program is core smithproxy component. Run without arguments, it loads `smithproxy.cfg` configuration 
file and starts to run on foreground.


## `sx_regencerts` command
Usage:
```
sx_regencerts
```
Signing CA authority generator script

Program doesn't take any arguments. It will check smithproxy certificate store and interactively
 asks user for desired actions.  

## `sx_cli` command
```
sx_cli [tenant_nane]
```
This is simple script connecting administrator to smithproxy CLI interface. It is tenant aware.  
If tenant table is present in the system, tenant_name is mandatory.

## `sx_network` command


## `sx_passwd` command

## `sx_download_ctlog` command

## `smithd` command