# Bind
[![CircleCI status](https://img.shields.io/circleci/project/github/uubk/bind/master.svg?style=shield)](https://circleci.com/gh/uubk/bind/tree/master)
![License](https://img.shields.io/github/license/uubk/bind.svg?style=popout)

Setup Bind9 as authoritative server with zones stored in LDAP. Tested on Debian 9.

## Note
The role assumes that you used e.g. [auth-server](https://github.com/uubk/auth-server) to pre-provision Kerberos principals, keytabs and the LDAP schema.

## Description
This role sets up Bind 9 as DNSSEC-validation resolver for loopback queries and as authoritative nameserver for everyone else. The data is stored in LDAP so that you can have multiple 'primary nameservers'.

## Configuration
| Name | Default value | Description |
| ---- | ------------- | ----------- |
| `bind_ldap_base` | `cn=dns,dc=example,dc=org` | The LDAP DN under which the DNS configuration will be stored. |
| `bind_ldap_url` | `ldaps://127.0.0.1` | URLs of LDAP servers (space separated) |
| `bind_ldap_krb_realm` | `EXAMPLE.COM` | Kerberos realm to use |
| `bind_ldap_ansible_url` | `ldaps://127.0.0.1` | URL of the LDAP server ansible is supposed to use |
| `bind_dns_ldap_zoneid` | `EXAMPLE` | ID of the LDAP zone section in bind's config |
| `bind_dns_zones` | `[]` | Array of DNS zones (see below) |
| `bind_dns_entries` | `[]` | Array of records in DNS zones (see below) |
| `bind_dnssec_keys` | `[]` | Array of DNSSEC keys for DNS zones (see below) |

An example configuration might look like this:
```
bind_dns_zones:
  - name: example.org
    attributes:
      idnsName: "example.org"
      idnsUpdatePolicy: grant EXAMPLE.ORG krb5-self * A;
      idnsAllowDynUpdate: "TRUE"
      idnsSecInlineSigning: "TRUE"
      idnsZoneActive: "TRUE"
      idnsSOAmName: "ns.example.org."
      idnsSOArName: "root.example.org."
      idnsSOAserial: 1
      idnsSOArefresh: 36000
      idnsSOAretry: 600
      idnsSOAexpire: 360000
      idnsSOAminimum: 400
      NSRecord:
        - ns.example.org.
      ARecord: 127.0.0.1
      dNSdefaultTTL: 3600

bind_dns_entries:
  - zone: example.org
    name: www
    attributes:
      ARecord: 127.0.0.1
      dNSTTL: 600

bind_dnssec_keys:
    # ZSK 2010
  - name: Kexample.org.+010+01532.key
    zone: example.org
  - name: Kexample.org.+010+01532.private
    zone: example.org
    # KSK 2010
  - name: Kexample.org.+010+04300.key
    zone: example.org
  - name: Kexample.org.+010+04300.private
    zone: example.org
```
This would publish `example.org` with (www.)example.org pointing to 127.0.0.1. The DNSSEC keys can be generated using for example (algorithm 10):
```
dnssec-keygen -a RSASHA512 -b 2048 example.org
dnssec-keygen -a RSASHA512 -b 2048 -f KSK example.org
```
For details on DNSSEC key rollover strategies, have a look at the official [BIND DNSSEC Guide](https://ftp.isc.org/isc/dnssec-guide/html/dnssec-guide.html#signing-maintenance-tasks). For details on the possible LDAP attributes, have a look at `files/dns.ldif` and at [the official documentation](https://github.com/freeipa/bind-dyndb-ldap). Note that most record types are actually defined in the COSINE schema.

## License
Apache 2.0
