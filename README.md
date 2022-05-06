# Ansible role to install and maintain the Bind9 nameserver on Debian

[![Build Status](https://github.com/systemli/ansible-role-bind9/workflows/Integration/badge.svg?branch=main)](https://github.com/systemli/ansible-role-bind9/actions?query=workflow%3AIntegration)
[![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-bind9-blue.svg)](https://galaxy.ansible.com/systemli/bind9/)

This role installs and configures the Bind9 nameserver on Debian.

Features:

* Support for configuring an authoritative nameserver for DNS zones and/or a DNS recursor
* Extensive DNSSEC support:
  * automatic KSK and ZSK key creation
  * automatic zone DNSSEC configuration
  * support to send DNSKEY/DS formatted output over XMPP
* Support for hidden primary and authoritative secondary configuration
* Support for so called "static" zones, i.e. zones defined uploading their raw .db bind file
* Validity check of zone files with named-checkzone
* Basic support for so called "dynamic" zones, i.e. defined from variables yaml variables sets

## Basic server configuration
### Master server
* set vars for your master server, for instance in `host_vars/master_name/vars/XX_bind.yml`, here with an example.com static zones and forwarder:
```yaml
bind9_authoritative: yes
bind9_zones_static: 
- { name: example.com , type=master }
bind9_forward: yes
bind9_forward_servers:
- 8.8.8.8
- 4.4.4.4
bind9_slaves:
- slave_ip_1
- slave_ip_2
- slave_ip_3
bind9_our_neighbors:
- slave_ip_1
- slave_ip_2
- slave_ip_3
```
* Place your BIND zone file in ansible directory (not in role directory): `files/bind/zones/db.example.com

### Slave servers

* set vars for your slave servers:

```yaml
bind9_zones_static: 
- { name: example.com, type: slave }
bind9_forward: yes
bind9_forward_servers:
- 8.8.8.8
- 4.4.4.4
bind9_masters:
- { name: master_name, addresses: [master_ip] }
bind9_recursor: our_network
```
### Dynamic zones
So called "dynamic" zones' records are defined through YAML ansible variable `bind9_zones_dynamic` which is parsed by [`bind/zones/db.template.j2`](templates/bind/zones/db.template.j2) template.
As there can be several zones, and zones definitions can be long, zones vars are worthly defined in a different vars' file, for instance `host_vars/master_name/vars/YY_zones.yml`, and  `bind9_zones_dynamic` can be splited in several variables, that can bie defined in specific files. In `YY_zones.yml` we may have:
```yaml
bind9_zones_dynamic: >
        {{ zones_my_domains
        | union ( zone_my_reverse_inaddr_arpa )
        | union ( zone_my_reverse_ip6_arpa ) }}

# bind9_zone_static:  zone files copied from `files/bind/zones/`

bind9_zones_static:
- name: static_dom.org
  type: master
- name: static_dom2.org
  type: master
- name: static_dom3.org
  type: slave
```
And in other vars files:
```yaml
zones_my_domains:
# This is the variables set for my domain
- name: dyn_domain.org
  type: master
  default_ttl: 600
  serial: 2022050501
  refresh: 1D
  retry: 2H
  expire: 1000H
  # NS and other pre-formatted records values must be given as full qualified domain names, with or without final dot, but not relative to the zone
  primary: ns1.dyn_domain.org         # Optional, if you don't define it, firs NS is taken 
  admin: postmaster.dyn_domain.org
  ns_records:
  - ns1.dyn_domain.org
  - ns2.dyn_domain.org
  # RR values are either relative to the zone, either with a final dot when outside.
  rrs:
  - {label: "@", type: MX, rdata: 10 mail}
  - {label: webmail, type: CNAME, rdata: mail}
  - {label: "@", type: A, rdata: 8.8.8.221}
  - {label: "@", type: AAAA, rdata: 2001:db8:6a::95}
  - {label: www, type: CNAME, rdata: webserver.dyn_domain.org.}
  - {label: mail, type: A, rdata: 8.8.8.222}
  - {label: mail, type: AAAA, rdata: 2001:db8:6a::22}
  - {label: webserver, ttl: 86400, type: A, rdata: 8.8.8.223}
  - {label: webserver, ttl: 86400, type: AAAA, rdata: 2001:db8:6a::23}
```

And similarly `zone_my_reverse_inaddr_arpa` and `zone_my_reverse_ip6_arpa` for IP reverse DNS resolution. Note that we adopted for generic NS records the terminology defined in [RFC 1034, Section 3.6](https://datatracker.ietf.org/doc/html/rfc1034#section-3.6)

* deploy role to your servers


## Dependencies

For the XMPP notification feature, `python-xmpp` needs to be installed.

## Role varibles

See `defaults/main.yml` for a list of role variables.

Testing & Development
---------------------

Tests
-----

For developing and testing the role we use Github Actions, Molecule and Vagrant. On the local environment you can easily test the role with

Run local tests with:

```
molecule test 
```

## License

This Ansible role is licensed under the GNU GPLv3.

## Author

Copyright 2017-2020 systemli.org (https://www.systemli.org/)
