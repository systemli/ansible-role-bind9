# Ansible role to install and maintain the Bind9 nameserver on Debian

[![Build Status](https://github.com/systemli/ansible-role-bind9/workflows/Molecule/badge.svg?branch=master)](https://github.com/systemli/ansible-role-bind9/actions?query=workflow%3AIntegration)
[![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-bind9-blue.svg)](https://galaxy.ansible.com/systemli/bind9/)

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
