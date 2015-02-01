
This runbook requires that you enable this option "error_on_undefined_vars=False", this allows us to have un-balanced hashes and set default variable values in default/main.yml.


# ansible-role-named

[![Build Status](https://travis-ci.org/midnightconman/ansible-role-named.svg?branch=master)](https://travis-ci.org/midnightconman/ansible-role-named)

 - Requires Ansible 1.5+
 - Compatible with most versions of RHEL/CentOS 6.x

## Installation

``` bash
$ ansible-galaxy install midnightconman.named
```

## Getting started

### Installing BIND (named) on Centos 6+

Installing BIND (named) and all required dependencies on Centos 6+ is very simple and can be done before configuration or individually on it's own: 

#### Install Only
``` bash
$ ansible-playbook -t install -i hosts, named.yml
```
#### Run the Whole Playbook
``` bash
$ ansible-playbook -i hosts, named.yml
```

### Example Playbook, Hosts, and Group Variables

#### Example Playbook
``` yml
---
- name: Actions Needed to Get Masters Into a Happy State
  hosts: named_masters
  remote_user: root

  roles:
    - named

- name: Actions Needed to Get Slaves Into a Happy State
  hosts: named_slaves
  remote_user: root

  roles:
    - named
```

#### Example Hosts
``` ini
[named_masters]
127.0.0.1

#[named_slaves]
#127.0.0.1
```

#### Example Group Variables

named_masters:
``` yml
---
named_acls:
  public_slaves:
    - 8.8.8.8
    - 9.9.9.9
  private_slaves:
    - 192.168.25.5
    - 192.168.25.6

named_zones:
# Notice the _ here, this makes ansible happy and is replaced later in config 
#  and zones automatically
  foo_com:
    type: master
    allow_transfer:
      - public_slaves
    ttl: 3000
```

named_slaves:
``` yml
---

named_zones_create_masters: False

named_zones:
# Notice the _ here, this makes ansible happy and is replaced later in config 
#  and zones automatically
  foo_com:
    type: slave
    master:
      - 7.7.7.7
```

**NOTE**: You must enable the ansible.cfg setting of "error_on_undefined_vars=False" or the environment setting for this role to function correctly. If you do not, you will see errors like:

```
'msg': "AnsibleUndefinedVariable: One or more undefined variables: 'dict' object has no attribute 'allow_notify'", 'failed': True
```

This role consists of installing tasks and configuring tasks, which are tagged with either 'install' or 'configure' and can be run individually or all together. This role has actions for creating a named.conf file, an included.conf file which will hold acls and zone includes, and dynamic zone files based on default or group variables.

## Configurables
 
There are quite a few configurables in this role, here is a summarized list of stock defaults (an up-to-date list can be found in defaults/main.yml):

``` yml
---
## Installation Options
named_conf_file_location: /etc/named.conf

# Make sure these are correct for your os
named_user: named
named_group: named
named_service_name: named

# Monit specific settings
named_monit_enable: False
named_monit_service_name: monit
named_monit_conf_directory: /etc/monit.d
named_pid_file: /var/run/named/named.pid
named_service_file: /etc/init.d/named

## Base Config Options

# Options Section
named_conf_listen_on_port: 53
named_conf_listen_on_interface:
  - 127.0.0.1
named_conf_listen_on_v6_port: 53
named_conf_listen_on_v6_interface:
  - ::1
named_conf_directory: /var/named
named_conf_dump_file: /var/named/data/cache_dump.db
named_conf_statistics_file: /var/named/data/named_stats.txt
named_conf_memstatistics_file: /var/named/data/named_mem_stats.txt
named_conf_allow_query:
  - any
#named_conf_allow_transfer: none
named_conf_recursion: no
named_conf_dnssec_enable: yes
named_conf_dnssec_validation: yes
named_conf_dnssec_lookaside: auto
named_conf_bindkeys_file: /etc/named.iscdlv.key
named_conf_managed_keys_directory: /var/named/dynamic

# Logging Section
named_conf_logging_channel: default_debug
named_conf_logging_file_directory: /var/log/named
named_conf_logging_file: named.log
named_conf_logging_severity: info
named_conf_logging_print_severity: yes
named_conf_logging_print_time: yes
named_conf_logging_print_category: yes
named_conf_logging_category_name: default
named_conf_logging_categories:
  - default_debug

named_conf_includes_directory: /etc/named

### No default acls or includes

## Master Settings
# This setting determines if zone files should be created in the declared
#  masters directory. Normally you wouldn't want to create these if you
#  are configuring a slave host.
named_zones_create_masters: True

### Zone Config Defaults
named_conf_zone_ttl: 21600
named_conf_zone_soa: foo.com. noc.foo.com.
named_conf_zone_refresh: 21600
named_conf_zone_retry: 600
named_conf_zone_expire: 86400
named_conf_zone_expire_min: 3000
```

Notice that you can specify default variables for dynamic zone file creation, this can allow for greatly reduced group_var files as you can only specify overrides for zones that require settings outside defaults.


## Facts

The following facts are accessible in your inventory or outside of this role.

- `{{ ansible_local.named.interfaces_ipv4 }}`
- `{{ ansible_local.named.interfaces_ipv6 }}`
- `{{ ansible_local.named.port_ipv4 }}`
- `{{ ansible_local.named.port_ipv6 }}`

