# Consul Cluster Role for Ansible

An Ansible Role for setting up Consul (https://consul.io)

## Organising hosts

This module requires 4 hosts groups: `consul`, `consul-bootstrap`, `consul-server` and `consul-client`:

* All Consul servers should belong under the `consul` group.
* A single cluster bootstrap server should belong under `consul-bootstrap` group
* The remaining servers should belong under `consul-server` group
* Any remaining client servers should belong under `consul-client` group

This role assumes a cluster of 5 nodes by default

### Example Hosts file

```
[consul:children]
consul-bootstrap
consul-server
consul-client

[consul-bootstrap]
consul1.example.com

[consul-server]
consul2.example.com
consul3.example.com

[consul-client]
consul4.example.com
consul5.example.com
```

## Variables

Here is an example variables file

```
---
consul:
  global:
    bind_adapter: ansible_eth0
    client_addr: 0.0.0.0
    datacenter: vagrant
    log_level: INFO
    data_dir: /var/lib/consul.d/
    encryption: <MY_ENCRYPTION_KEY>
  bootstrap:
    expect: 3
```

This will create a 5 node cluster - the web UI will be available on any client node address.

## Generating Encryption Keys

The data and communication across the gossip protocol is encrypted. All nodes must use the same encryption key which is a random 16 byte string. You can generate this will the `consul keygen` command or the following if `consul` isn't available on your machine:

`dd if=/dev/urandom of=/dev/fd/1 bs=1 count=16 | base64`

Add the resulting key to your variables prior to running the role.

## Adding services

Service files can be written by adding a dictionary of services to the variables file. For example to add an Apache service:

```
consul:
  services:
    apache:
      tags:
        - web-tier
        - apache
      port: 80  
```

## Adding Healthchecks

Adding healthchecks can be done in much the same way - for example here is a script health check for the above service

```
checks:
    - name: apache
      args:
        - curl
        - localhost
      interval: 10s
```

## Example Task with role

```
- name: Consul | Cluster Setup
  hosts: consul
  roles:
    - edr.consul
```
