# rotate-tlsa
This tool is a Bash script that installs TLSA keys into a DNS zone using RFC2136 with bind's nsupdate command. It can be run immediately after "danebot renew"
(from tlsaware danebot). If the key has changed it will rotate the current key to previous after retiring any previous key's TLSA records, so that there should
never be more than one TLSA record in the system for any given name.

To install, copy rotate-tlsa into `/usr/local/sbin` and `rotate-tlsa.service` to `/lib/systemd/system` (this assumes that you are using systemd and have danebot
set up to launch via systemd).

Configuration is done with files in `/usr/local/etc/rotate-tlsa`. For each TLSA record watched in `/etc/letsencrypt/staging/*/dane-ee`, there should be a configuration
file in `/usr/local/etc/rotate-tlsa` with a name of the form `domain.port.protocol`. For example, if a `dane-ee` file has an entry:

```
1 _25._tcp.example.com
```
the corresponding configuration file should be in `/usr/local/etc/rotate-tlsa/example.com.25.tcp`.

The configuration file is a bash script that provides for setting two variables:

1. `credsfile` (mandatory) - specify the location of an shell script containing RFC2136 credentials. If your credentials file for certbot does not include spaces before or after the equal sign, you can effectively use the same credentials file you use for certbot. It must set all of `dns_rfc2136_server`, `dns_rfc2136_port`, `dns_rfc2136_name`, `dns_rfc2136_secret`, and `dns_rfc2136_algorithm`
3. `ttl` (optional) - specify the TTL to be used in the TLSA records. Default: 86400.

Example:

```
credsfile=/etc/bind/rfc2136-creds.ini
ttl=3600
```
