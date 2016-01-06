#openvpn-ddns

Maintain dns records for connecting openvpn clients

# Usage

The following text assumes, that an openvpn server with client certificates,
as described in the [openvpn documentation](https://openvpn.net/index.php/open-source/documentation/miscellaneous/77-rsa-key-management.html)
and a nameserver which supports update via nsupdate such as [bind](https://www.isc.org/downloads/bind/).

1. Install Ruby and nsupdate

2. Clone Project

```
$ cd /etc/openvpn
$ git clone https://github.com/Mic92/openvpn-ddns.git ddns
```

3. Edit Configuration

```
$ cp /etc/openvpn/ddns/openvpn-ddns.json.example /etc/openvpn/openvpn-ddns.json
```

In case you have multiple openvpn server you can also create a configuration per
profile:

```
$ cp /etc/openvpn/ddns/openvpn-ddns.json.example /etc/openvpn/server1.openvpn-ddns.json
$ cp /etc/openvpn/ddns/openvpn-ddns.json.example /etc/openvpn/server2.openvpn-ddns.json
```

where `server1` or `server2` is the name of the openvpn configuration files without the
`.config/.ovpn` extension.

the configuration takes the following following keys:

- **name_server**: 
  - string
  - required
  - hostname or ip address to nameserver                                                        |
- **nsupdate_executable**:
  - string
  - optional
  - path or name of nsupdate (Defaults to "nsupdate")                                           |
- **private_key**: 
  - string
  - optional
  - If set, this will be used by nsupdate to authenticate against nameserver, use the format `algorithm:keyname key`, where `keyname` is the name used in nameserver configuration and `algorithm` the used TSIG key algorithm.
  - example: `hmac-sha512:ddns-key NTc1ODVmNDk5NzgwMDgyODQ2ZTAzMGNlZmI0YTkwN2M5ZTg1MzNiN2UxMWQyNjZhNjg2YWQ1MDc4Y2NlZjU0Mw==`
- **reverse_zones**
  - array
  - required
  - list of reverse zones to add a PTR recored in
- **zones**:
  - array
  - required
  - list of zones, you want the comon name to be added in

A dnssec-key can be obtained like this:

```
$ ddns-confgen -q -a hmac-sha512 -k openvpn
key "openvpn" {
        algorithm hmac-sha512;
        secret "NTc1ODVmNDk5NzgwMDgyODQ2ZTAzMGNlZmI0YTkwN2M5ZTg1MzNiN2UxMWQyNjZhNjg2YWQ1MDc4Y2NlZjU0Mw==";
};
```

4. Run openvpn

For adding/removing clients, add this to the server config

```
learn-address /etc/openvpn/ddns/openvpn-ddns
script-security 2
```

For adding the ip of the server itself:
```
up /etc/openvpn/ddns/openvpn-ddns
script-security 2
```

Down is not supported, as it's unlikely that a vpn server is removed entirely...

At the moment only `tun` mode of openvpn is supported.
