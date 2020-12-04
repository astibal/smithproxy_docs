# Setting up CA in smithproxy

## Currently installed CA 

To check proxy currently configured certificate authority, run (snap uses `smithproxy.certinfo`):
```
root@b760531c0654:/app# sx_certinfo_ca 
[+] Smithproxy certificate info: 

  Path: /etc/smithproxy/certs/default/ca-cert.pem

  Signature Algorithm: ecdsa-with-SHA256
  Issuer: CN = Smithproxy Root CA, O = Smithproxy Software, C = CZ
  Validity
      Not Before: Feb 20 12:21:22 2020 GMT
      Not After : Apr 21 12:21:22 2020 GMT
  Subject: CN = Smithproxy Root CA, O = Smithproxy Software, C = CZ
  Subject Public Key Info:
      Public Key Algorithm: id-ecPublicKey

[+] CA certificate (load to systems as trusted CA certificate): 

-----BEGIN CERTIFICATE-----
MIICDDCCAbKgAwIBAgIKLKoCO1gU+KhYPjAKBggqhkjOPQQDAjBIMRswGQYDVQQD
DBJTbWl0aHByb3h5IFJvb3QgQ0ExHDAaBgNVBAoME1NtaXRocHJveHkgU29mdHdh
cmUxCzAJBgNVBAYTAkNaMB4XDTIwMDIyMDEyMjEyMloXDTIwMDQyMTEyMjEyMlow
SDEbMBkGA1UEAwwSU21pdGhwcm94eSBSb290IENBMRwwGgYDVQQKDBNTbWl0aHBy
b3h5IFNvZnR3YXJlMQswCQYDVQQGEwJDWjBZMBMGByqGSM49AgEGCCqGSM49AwEH
A0IABJ10gcOSo3O5I5zhKmutlKHE6FQSTILFzb4dH0rL0mxQxpbT1lrsy6Tb29yQ
nFl8oEKjoJMKQBvveb9WJ63Od9SjgYMwgYAwHQYDVR0OBBYEFP8mzHeUk10D7CvW
fNqJaE4pCnzcMB8GA1UdIwQYMBaAFP8mzHeUk10D7CvWfNqJaE4pCnzcMB0GA1Ud
EQQWMBSCElNtaXRocHJveHktUm9vdC1DQTAPBgNVHRMBAf8EBTADAQH/MA4GA1Ud
DwEB/wQEAwIBBjAKBggqhkjOPQQDAgNIADBFAiAmgthT0uaJnkFJrPCKUWXpb3RE
ZUL5QePGkaQbu7x6egIhAMmkx7/oZIY19vklzpYvhF55UYm/sBeOZau/Ec6+LaZ+

```

In fresh smithproxy installation you will default and expired certificate. This is intentional.
Please generate your own, new CA certificate. Trusting signatures made by CA certificate issued by 
strangers is not good practice (in common language: it's stupid and insecure).   


## Generating new signing CA for our proxy 

Run `sx_regencerts` (by *root* or via `sudo`) command to generate a new certificates. If instructed 
to generate new CA, script will also generate a new *default server certificate* and *portal certificate*.

Follow questions and modify any option you like.

```
root@b760531c0654:/app# sx_regencerts 
Do you want to check and generate new certificates?
   - Note: you can modify attribute values in file: /etc/smithproxy/certs/default/sslca.json
   -  [No/Yes/Dry/Enforce]? Yes
== Checking installed certificates ==
== Checking CA cert ==
   - Default CA delivered by packaging system has been detected.
   ==> Do you want to generate your own CA? [yes/no]? 
   ==> Lifetime in days [60/Other]? 
   - i: using ttl 60
== checking default server cert ==
    warning: certificate /etc/smithproxy/certs/default/srv-cert.pem expires, or already expired
   - New default server certificate will be generated (new CA)
== checking portal cert ==
    certificate /etc/smithproxy/certs/default/portal-cert.pem valid.
   - New portal certificate will be generated (new CA)
== Execute ==

   ==> Which CA type you prefer? [rsa/ec]? ec

   - New Certificate Authority:
     - generating new CA == 
   - GENERATED

   - Server certificate:
     - generating server certificate == 
   - GENERATED

   - Portal certificate:
     - generating server certificate == 
   - cfg portal address is IP
   - GENERATED

== Finished ==

   - !!! New CA certificate: 
-----BEGIN CERTIFICATE-----
MIICDTCCAbOgAwIBAgILAIQf8qD/nzN6nXkwCgYIKoZIzj0EAwIwSDEbMBkGA1UE
AwwSU21pdGhwcm94eSBSb290IENBMRwwGgYDVQQKDBNTbWl0aHByb3h5IFNvZnR3
YXJlMQswCQYDVQQGEwJDWjAeFw0yMDEyMDIwOTMzMzJaFw0yMTAyMDEwOTMzMzJa
MEgxGzAZBgNVBAMMElNtaXRocHJveHkgUm9vdCBDQTEcMBoGA1UECgwTU21pdGhw
cm94eSBTb2Z0d2FyZTELMAkGA1UEBhMCQ1owWTATBgcqhkjOPQIBBggqhkjOPQMB
BwNCAAS+A+y7jmmITHHRGYT5LFFD0C2Qm16jZSgUUza5bZxeBAv00nNK4q4XSJrk
jh0mivPTrntEDOoZz+cwnzv/UpGco4GDMIGAMB0GA1UdDgQWBBSVQTdOkw0w5nYs
aDmlU+O2idxaDzAfBgNVHSMEGDAWgBSVQTdOkw0w5nYsaDmlU+O2idxaDzAdBgNV
HREEFjAUghJTbWl0aHByb3h5LVJvb3QtQ0EwDwYDVR0TAQH/BAUwAwEB/zAOBgNV
HQ8BAf8EBAMCAQYwCgYIKoZIzj0EAwIDSAAwRQIgC1+gggciJItOq4TtLr1tFRdR
/sCD/ngkkns6xVlLuiwCIQCTl9mjvtglaKDcwAfnpsFH0vKPMyZFzINFqxa2RVeZ
Dg==
-----END CERTIFICATE-----

```

> !!! tool won't save certificate correctly if not run by privileged user.

  

> Note: smithproxy has to be restarted to make it use the new CA certificate.

## Certificate location on filesystem

All certificates are located in `/etc/smithproxy/certs/`:

```
root@b760531c0654:/app# ls /etc/smithproxy/certs/default/ -la
total 56
drwxr-xr-x 1 root root 4096 Dec  3 09:12 .
drwxr-xr-x 1 root root 4096 Nov 26 00:25 ..
-rw-r--r-- 1 root root  774 Dec  3 09:33 ca-cert.pem
-rw-r--r-- 1 root root  227 Dec  3 09:33 ca-key.pem
-rw-r--r-- 1 root root 1684 Nov 26 00:25 cl-cert.pem
-rw-r--r-- 1 root root 1679 Nov 26 00:25 cl-key.pem
-rw-r--r-- 1 root root 1155 Dec  3 09:57 portal-cert.pem
-rw-r--r-- 1 root root  120 Nov 26 00:25 portal-gen.info
-rw-r--r-- 1 root root 1679 Dec  3 09:57 portal-key.pem
-rw-r--r-- 1 root root 1115 Dec  3 09:33 srv-cert.pem
-rw-r--r-- 1 root root 1679 Dec  3 09:33 srv-key.pem
-rw-r--r-- 1 root root  832 Dec  3 09:57 sslca.json

```
Note: `sslca.json` is simple config file used by `sx_regencerts`. If you are not happy with the CA certificate
attributes, you can modify `sslca.json` and rerun the tool. It will follow the settings from json config.
This small .json file doesn't serve to any other purpose in the proxy.

Note: `portal-gen.info` - remove this file to disable automatic portal certificate regeneration on service startup.

## Default server certificate

When new CA is generated, also new, *default server certificate* is generated. 
This certificate is however not used in TLS connections, because it doesn't contain
any real target certificate properties (ie CN, SAN, ... ). 

Default server certificate purpose is to serve as public key-pair template. In other words,
its private and public keys are used to generate properly filled in signing request which is
later signed by smithproxy root CA. New certificate is then delivered to client, having properties as desired (spoofed certificate details).

In other words, you will see correctly spoofed certificate, with `RSA` or `EC` keys of default server certificate.


## Default portal certificate

Portal certificate is an end entity certificate used by authentication portal. It is **regenerated on each
 service restart** to reflect changes in hostname and IP addresses (this can be turned off, see previous chapter).
Auto-generated portal certificate is signed by Smithproxy root CA, and uses DNS SANs and IP SANs to make as wide
 match on "lazy installation" as possible.

The flow is:
1) client tries to reach internet, but authentication is configured
2) smithproxy resigns the original server certificate (using def server cert keypair), but content is replaced
   with HTTP redirection to a portal (configurable fqdn and port)
3) client follows the redirection and gets https logon screen with a portal certificate   

This is however a bit tricky. The portal certificate must match the hostname in the client (typically a browser). The
 redirection link in step 2) therefore must match the certificate CN, or SAN.    

You have couple of options to achieve this

#### Portal redirection with automatically generated portal certificate 

Set the portal fqdn in smithproxy config to smithproxy hostname and rely on startup portal regeneration. Hostname
 must be in this case resolvable by DNS on client, or it can be even IP address. 
```
settings = 
{
  // ...
  auth_portal = 
  {
    address = "192.168.254.1"
    address6 = "[2002:5f8f:81be:2::1]"
    http_port = "8008"
    https_port = "8043"
    // ...
  }

```

#### Portal redirection using custom portal certificate

You can disable portal certificate auto-generation, and use your own one:

```
settings = 
{
  // ...
  auth_portal = 
  {
    address = "sx.simpsons.com"
    address6 = "sx6.simpsons.com"
    http_port = "8008"
    https_port = "8043"
    ssl_key = "custom-portal-key.pem"
    ssl_cert = "custom-portal-cert.pem"
  }

```

In this case you see custom portal cert filenames. If you keep original `portal-*.pem`, they would be replaced by
 auto-generation on next startup.
 You can disable portal cert auto-generation by removing `portal-gen.info` file.

