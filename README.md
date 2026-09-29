# rotate-tlsa - install and update TLSA records to support DANE for keys managed by tlsaware danebot
This tool is a Bash script that installs TLSA records into a DNS zone using RFC2136 with bind's nsupdate command. It can be run immediately after "danebot renew"
(from tlsaware danebot). If the key has changed it will rotate the current key to previous after retiring any previous key's TLSA records, so that there should
never be more than two TLSA records in the system for any given name.

To install:
1. copy `rotate-tlsa` into `/usr/local/sbin` and `rotate-tlsa.service` to `/lib/systemd/system` (this assumes that you are using systemd and have danebot set up to launch via systemd).
2. install `certbot`, `danebot` and the `tlsa` command (in the `hash-slinger` package on Debian) if not already installed.

Configuration is done with files in `/usr/local/etc/rotate-tlsa`. For each TLSA record watched in `/etc/letsencrypt/staging/*/dane-ee`, there should be a configuration
file in `/usr/local/etc/rotate-tlsa` with a name of the form `domain.port.protocol`. For example, if a `dane-ee` file has an entry:

```
1 _25._tcp.example.com
```
the corresponding configuration file should be in `/usr/local/etc/rotate-tlsa/example.com.25.tcp`.

The configuration file is a bash script that provides for setting two variables:

1. `credsfile` (mandatory) - specify the location of a Bash shell script providing RFC2136 credentials. If your credentials file for certbot does not include spaces before or after the equal sign, you can effectively use the same credentials file you use for certbot. It must set all of `dns_rfc2136_server`, `dns_rfc2136_port`, `dns_rfc2136_name`, `dns_rfc2136_secret`, and `dns_rfc2136_algorithm`
2. `ttl` (optional) - specify the TTL to be used in the TLSA records. Default: 86400.
3. `extra_view_keynames` (optional) - specify an array of additional key names for registering updates. This is useful for sites with multiple views in BIND (for an example, and internal view and an external view). By setting up multiple keys in the BIND configuration, each with the same secret, the same TLSA records can be propagate to both the internal and external views by using ACLs conditioned on the key. This will be necessary where the host danebot runs on normally accesses the internal view (so danebot sees the internal view when it checks for the TLSA records being present).

Example 1 - configuration limiting the TTL to 1 hour:

```
credsfile=/etc/bind/rfc2136-creds.ini
ttl=3600
```

Example 2 - configuration that needs to set the TLSA records in an additional view

```
credsfile=/etc/bind/rfc2136-creds.ini
extra_view_keynames=( dane-key.internal.example.com )
```
