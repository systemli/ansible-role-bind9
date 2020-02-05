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

```yml
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
# bind9_zone_dynamic: zone files created from template (see
#         `templates/bind/zones/db.template.j2` for yaml structure)
# bind9_zone_static:  zone files copied from `files/bind/zones/`
bind9_zones_dynamic: []
bind9_zones_static: []

# Send DNSSEC ZSK in DNSKEY and DS format over XMPP after it got created
bind9_dnssec_notify_xmpp: no
bind9_dnssec_notify_xmpp_user: user@jabber.example.org
bind9_dnssec_notify_xmpp_password: insecure
bind9_dnssec_notify_xmpp_rcpt: admin@jabber.example.org
bind9_dnssec_notify_xmpp_host: "{{ ansible_jabber_host|default('localhost') }}"

# Install monit file for bind9 named
bind9_monit_enabled: no

bind9_packages:
    - bind9
    - dnsutils
    - haveged

# Directory for bind9 files templates
bind9_templates: ""
# The default value takes templates form the {{ role_path }}/templates/ directory of the role 
# You can set your own templates, for example with: 
# bind9_templates: "{{ playbook_dir }}/host_vars/<my_host>/templates/"

# Logging
bind9_named_logging: False
bind9_log_path: /var/log/bind
bind9_log_severity: warning  # critical | error | warning | notice | info | debug [ level ] | dynamic
bind9_log_versions: 3
bind9_log_size: 60m           # Time units

bind9_log_categories:
  - name: default
    destination: bind_log
  - name: update
    destination: bind_log
  - name: update-security
    destination: bind_log
  - name: security
    destination: default_syslog
  - name: queries
    destination: bind_log
  - name: lame-servers
    destination: 'null'
```

Testing & Development
---------------------

Tests
-----

For developing and testing the role we use Travis CI, Molecule and Vagrant. On the local environment you can easily test the role with

Run local tests with:

```
molecule test 
```

## License

This Ansible role is licensed under the GNU GPLv3.

## Author

Copyright 2017-2018 systemli.org (https://www.systemli.org/)
