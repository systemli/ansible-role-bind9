# Ansible role to install and maintain the Bind9 nameserver on Debian

[![Build Status](https://travis-ci.org/systemli/ansible-role-bind9.svg?branch=master)](https://travis-ci.org/systemli/ansible-role-bind9) [![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-bind9-blue.svg)](https://galaxy.ansible.com/systemli/bind9/)

This role installs and configures the Bind9 nameserver on Debian.

Features:
* Support for configuring an authoritative nameserver for DNS zones and/or
  a DNS recursor
* Extensive DNSSEC support:
  * automatic KSK and ZSK key creation
  * automatic zone DNSSEC configuration
  * support t osend DNSKEY/DS formatted output over XMPP
* Support for hidden primary and authoritative secondary configuration
* Basic support for dynamic creation of zone files from variables

## Dependencies

For the XMPP notification feature, `python-xmpp` needs to be installed.

## Role varibles

```
# User and group for bind
bind9_user: bind
bind9_group: bind

# Listen on IPv6 interfaces
bind9_ipv6: yes

# Run bind as a DNS recursor?
bind9_recursor: no

# Run bind as authoritative nameserver?
bind9_authoritative: no

# Setup DNSSEC for recursor and zones?
bind9_dnssec: no

# Run bind as a hidden master (i.e. limit queries to our_networks)
bind9_hidden_master: no

# Only notify nameservers from also-notify, not from the zone NS records.
# Necessary to keep traffic between nameservers in private network.
bind9_notify_explicit: no

# Default zone type
bind9_zone_type: master

# Permitted hosts/networks for recursion (when configured as recursor)
bind9_our_networks:
  - localhost
  - localnets

# Permitted hosts/networks for zone transfers
bind9_our_neighbors:
  - localhost
  - localnets

# Install custom rndc.key
bind9_rndc_algorithm: hmac-md5
#bind9_rndc_key:

# Global primaries for all zones (if configured as secondary)
#bind9_masters:
#  - name: ns-primary
#    addresses:
#      - 1.2.3.4

# Global secondaries for all zones (if configured as primary)
#bind9_slaves:
#  - 1.2.3.4

# DNS Zones
# bind9_zone_dynamic: zone files created from template, see `templates/bind/zones/db.template.j2` for yaml structure
# bind9_zone_static:  zone files copied from `files/bind/zones/`
bind9_zones_dynamic: []
bind9_zones_static: []

# Send DNSSEC ZSK in DNSKEY and DS format over XMPP after it got created
bind9_dnssec_notify_xmpp: no
bind9_dnssec_notify_xmpp_user: user@jabber.example.org
bind9_dnssec_notify_xmpp_password: insecure
bind9_dnssec_notify_xmpp_rcpt: admin@jabber.example.org

# Install monit file for bind9 named
bind9_monit_enabled: no
```

## License

This Ansible role is licensed under the GNU GPLv3.

## Author

Copyright 2017-2018 systemli.org (https://www.systemli.org/)
