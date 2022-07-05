# Ansible role to install and maintain the Bind9 nameserver on Debian

[![Build Status](https://github.com/systemli/ansible-role-bind9/workflows/Integration/badge.svg?branch=main)](https://github.com/systemli/ansible-role-bind9/actions?query=workflow%3AIntegration)
[![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-bind9-blue.svg)](https://galaxy.ansible.com/systemli/bind9/)

This role installs and configures [BIND9](https://www.isc.org/bind/) on Debian to set a Name Server.

Features:

* Support for configuring BIND9 according to a set of template: 
  * default templates can implement an authoritative nameserver for DNS zones and/or a DNS recursor with or without forwarders,
  * a set named `strict_authoritative` for a secure and easy configuration of a set of authoritative servers, primaries and secundaries,  
* Extensive DNSSEC support:
  * automatic KSK and ZSK key creation
  * automatic zone DNSSEC configuration
  * support to send DNSKEY/DS formatted output over XMPP
* Support for hidden primary and authoritative secondary configuration
* Support for so called "static" zones, i.e. zones defined uploading their raw .db bind file
* Validity check of zone files with named-checkzone
* Basic support for so called "dynamic" zones, i.e. defined from variables yaml variables sets

## Basic server configuration

Lest's start by a simple but complete configuration of two servers:

### Master server

* set vars for your master server, for instance in `host_vars/master_name/vars/XX_bind.yml`, here with an example.com static zone and forwarder:
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
* Place your BIND zone file in ansible directory (not in role directory): `files/bind/zones/db.example.com`. The role will check the validity of this file.

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
* deploy role to your servers!

## Static zones and Dynamic zones

In previous example, zones' ressource records are defined by a classic BIND9 zone's file, which validity is checked, but that you have to maintain. These are the so called "static zones", raw defined by a `db.<zone_name>` file.

So called "dynamic" zones' files are built form ansible variables. Their ressource records are defined through YAML ansible structure `bind9_zones_dynamic` which is parsed by [`bind/zones/db.template.j2`](templates/bind/zones/db.template.j2) template.

As there can be several zones, and zones' definitions can be long, zones' vars are worthly defined in a different vars' file, for instance `host_vars/master_name/vars/YY_zones.yml`, and  `bind9_zones_dynamic` can be split in several variables, that can be defined in specific files. In `YY_zones.yml` we may have:
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
And in other vars' files:
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
  master: ns1.dyn_domain.org         # Optional, if you don't define it, firs NS is taken 
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

And similarly `zone_my_reverse_inaddr_arpa` and `zone_my_reverse_ip6_arpa` for IP reverse DNS resolution. Note that for generic NS records we adopted the terminology defined in [RFC 1034, Section 3.6](https://datatracker.ietf.org/doc/html/rfc1034#section-3.6)


## Configurable templates' set

Basically the role builds bind9 configuration, i.e. `/etc/bind/named.conf.*` files, as well as zone definition files, which are placed in `/etc/bind/zones/` directory.

Configuration is based on a set of templates, and the role can handle several ones. Presently two sets of templates are proposed: 
* the default one, a general purpose set of templates that has evolved with the role, 
* a "_strict authoritative_" NS templates' set, that: 
  * denies by default any query, recursion or transfer, and only allows queries from any and transfers from slaves for zones the server is authoritative on.
  * automates, for each zone, the inclusion of secundary NS servers and also-notify IPs in the allow-transfer permissions. 

Templates' set is defined by variable `bind9_templates`. You should set it to the absolute path or the relative path to the `templates/` directory of the role. For [strict authoritative NS config](templates/strict_authoritative/), define in your vartiables: 
```yaml
bind9_templates: strict_authoritative/
```
Several variables of the role define options and values in the set of templates you use. Note that the same role's variable of the role may have slightly different meanings, or no meaning at all, depending on the choosen set of templates.

You can develop your own set of templates and set, for instance: 
```yaml
bind9_templates: "{{ playbook_dir }}/host_vars/<my_host>/templates/"
```
PRs with good BIND9 configs templates are welcome!

### `strict_authoritative` templates' set

[This templates' set](templates/strict_authoritative) is designed to configure strict [Authoritative Name Servers](https://bind9.readthedocs.io/en/latest/chapter3.html#config-auth-samples), primaries or secondaries:
* queries are refused except for the zones we are authoritative for,
* transfers are selectively set by zone and no recursion at all, 
* when we answer, we give th same answer to the whole internet (for public internet zones, baroque configurations that restrict answers or, worse, give different answers to different clients, such as with views, are bad ideas that break the internet, considering DNS is a core part of it),
* by default, zone transfer are allowed, zone by zone, to secundaries and also-notify elements
* customizations should allow to configure all kind of sets of primaries or secudaries, visible or hideden.

Therefore, when you set `bind9_templates: strict_authoritative/`, in `options` BIND configuration section, these templates always set: 
```
recursion no;
allow-query { none; };
allow-transfer { none; };
allow-recursion { none; };
```

Default configuration values are set with `bind9_<parameter>` role variables, that can be overwritten for each zone with specific values in the `.<parameter>` field in the corresponding zone's element in `bind9_zones_static` or `bind9_zones_static`. 

Depending on the parameter considered, templates implement the default value either in the `options` section of BIND config files (`notify`, `also-notify`), either picking default values and set them in the zone's configuration section (`allow-query`, `allow-transfer`). With ACLs and setting parameter values per zone, the role can handle all sort of particular cases for some zones.  

The YAML structure of ther variables allows to overcome and unify (at least for simplified configurations the role permits) BIND's management of two kind of lists: masters and acl. Templates take advantage of this characteristic to automatically configure secundary NS IPs and also-notify IPs in the zone's `allow-transfer` configuration directive for zones we are master for. Zone parameters are defined to extend or to overwrite this list of allow-transfer. 

### `strict_authoritative` use cases

* `bind9_masters` and `bind9_slaves` should be enough for standard internet zones and if you have the same set of NS authoritative servers for all your zones. You don't have to care about `allow-transfer` for slaves in the master server: the role does the job.
* Use `bind9_also_notify` if you have some hidden NS servers. Don't worry neither about `allow-transfer` to those hosts, the role also does the job.
* If some of your zones have specific configuration: 
  * `bind9_masters_extra` can help you for different sets of masters for your secundary zones,
  *  `bind9_acl`, along with specific values per zone set in `bind9_zones_static` or `bind9_zones_dynamic`, will help to set all kind of transfers, notifications and even to restric queries for eventual private zones.  
* zone by zone, the templates also takes care of including slaves and also-notify hosts for allow-transfer. 

### Role's variables for `strict_authoritative` templates

Explicitely, you can use the following variables to configure your NS server and its zones: 
* `bind9_masters`: default primary (or master) NS servers for zones we are secondary (or slave) for.
  ```yaml
  bind9_masters:
    my_primariy:
    - IPv4_1
    - IPv6_1
    my_fault_back_primary:
    - IPv4_1
    - IPv6_1
  ```
  With these lists the template builds the [primaries' or masters' list(s)](https://bind9.readthedocs.io/en/latest/reference.html?highlight=primaries%20list#primaries-statement-grammar) to be used by default as `masters` for zones we are slave for, when masters are not specifically set for the zone. Note that masters' list can only other masters' lists names or individual IPs. IPv4 or IPv6, but not network IPs range, ending with a / and a mask length. 

* `bind9_masters_extra` is a similar structure that also sets primaries' lists, but which are not the role's default values for zones declared.

  __Magic-mix of acls and masters' lists__: Thanks to the similarity of these YAML structure with `bind9_acl` variable hereafter, for each element of `bind9_masters` and `bind9_masters_extra` the templates build, moreover the primaries' list, an [Access Control List (ACL)](https://bind9.readthedocs.io/en/latest/chapter6.html#access-control-lists) with the same name and content. Therefore, contrarily than when working directly on BIND9 configuration files, we can use masters' list names, not only in [the variables that define] `masters` or `also-notify` clauses of a zone, but also in [the variables that define] clauses which work with ACLs, such as `allow-transfer` or `allow-query`. Taking advantage of this trick, the role can automatically include a similar content into a notify-also clause (which requires a masters' list) as well as in the allow-transfer ones (which require an ACL). See hereafter.

* `bind9_acl` has a quite similar structure with keywords and list of IPs but, in BIND configuration, it builds global [Access Control lists](https://bind9.readthedocs.io/en/latest/chapter6.html#access-control-lists), to be used in appropriate parameters, particularly in the `allow-xxx` per zone. Note that `bind9_acl` can contain not noly IPs (IPv4 and IPv6) but also network IPs ranges, ending with a / and a mask length, as well as a literal reference to recursively include another ACL. As ACLs are already defined for masters, so do not use for ACL a name you already used in a masters' list. 

* `bind9_slaves` defines the default secundary or slave NS servers for zones we are primary or master for. It's a list of IPs, or eventually ACLs defined with previous variable. For zones we are primary for, templates also include these secondaries IPs in those allowed to transfer the zone, except if the `.slaves` or the `.allow_transfer` parameter hereafter is defined for the zone.

* `bind9_notify` can take the values `master-only`, `explicit`, `yes` or `no`. It defines the defaut behavior for notification, wich is set in [`options` section](templates/strict_authoritative/bind/named.conf.options.j2#L41). Letting `bind9_notify` undefined doesn't set the directive and therefore leads to BIND's default behavior, i.e. `notify: yes`. We don't want to overwrite BIND's default behavior, but for most purposes of these templates, we recommend to set notify to `master-only`. 

* `bind9_also_notify` is a list that defines the global `also-noitfy`. As such, it can contain IPs or masters' lists, that can be set with the variables `bind9_masters` and `bind9_masters_extra` hereabove. Except if `.also_notify` or `.allow_transfer` is explicitely set for the zone, templates take advantage of the masters' lists _and_ ACLs built for `bind9_masters` and `bind9_masters_extra` to include the content of `bind9_also_notify` in the zone's `allow-transfer` directive, what would not be possible with BIND's syntax and grammar alone. Names are interpreted as masters' lists in `also-noitfy` clauses and as ACLs in `allow-transfer` ones, but the templates have defined both with the same name and content. Globally or per zone, also-notify option is useful when you have hidden NS servers.

* `bind9_also_allow_transfer` is a variable that can contain a list of IPs, network ranges of IPs and ACL names defined with `bind9_acl`, that will be added, by default for zones we manage, to the `allow-transfer` inferred by the role, which already includes secundaries and also-notify elements, as documented hereabove. This option may be useful for some very strange configuration, where NS servers are not notified but should be allowed to transfer zones. 

* `bind9_allow_transfer` is a variable that can contain a list of IPs, network ranges of IPs and ACL names, defined with `bind9_acl`. If defined, it will cancel the mechanism previously defined to include secundaries and also-notify in allow-transfer zone's clause. Except if `.allow_transfer` is defined for the zone, the content of `bind9_allow_transfer` and only this will populate the `allow-transfer` zone's clause. 

Zone by zone, in `bind9_zones_static` as well as in `bind9_zones_dynamic` list's elements, the following zone configuration parameters can be defined: 

* `.allow-query`: a list of IPs and/or acls to restrict the clients that can ask for the zone. Default value is `any`. (breaks the internet if you publish an NS record: use with care, only for private zones)
* `.type`: that defines de BIND9 type of zone: `master`, `slave`, `forward`, `stub`
* `.masters`: can be defined for the zones we are slave for. You can put there anything that BIND9 accepts but, to take advantage of this role to avoid data duplication, we recommend to declare all your NS hosts in `bind9_masters` and `bind9_masters_extra`, and reference them by name.
* `.slaves`: can be defined for zones we are master for. It has a similar structure than `bind9_slaves`, that it will overwrite for the zone. The hosts it contains will be allowed for transfer, except if the parameter `.allow_transfer` hereafter is defined. 
* `.notify`: `explicit`, `yes` or `no` (`master-only` has no really useful for a single zone). Sets notification behavior for the zone,
* `.also_notify`: can be defined zone by zone we are master for, with a similar structure than `bind9_also_notify`, that it will overwrite for the zone. and the hosts it contains will be allowed for transfer, except if the parameter `.allow_transfer` hereafter is defined. 
* `.also_allow_transfer`: is a list similar to an ACL, that will overwrite `bind9_also_allow_transfer` if defined, wich will be added to the `allow-transfer` statement content inferred by the role as described hereabove.
* `.allow_transfer`: is a list similar to an ACL, that will overwrite `bind9_allow_transfer` for the zone. Its content and only this will populate the `allow-transfer` zone's clause.

Finally, note that this set of templates no longer uses several variables that use the default templets, such as `bind9_recursor` or `bind9_our_neighbors`, for instance. However, `bind9_authoritative`, whilst not used in the templates is tested by the role as conditionnal of several tasks. Therefore, it _must_ be set to true (as does de defaults' definition). 

### Default templates and other roles's variables

Default templates have evolved with the role and try to set all kind of BIND9 configuration, as authoritative, resolver or forwarder. However, as initially there was no per zone BIND configuration, they have a different logic.   

See also [`defaults/main.yml`](defaults/main.yml) for a list of role's variables, including those which are used by default templates and not by `strict_authoritative/` ones.


## Dependencies

For the XMPP notification feature, `python-xmpp` needs to be installed.

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
