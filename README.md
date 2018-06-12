# Consul Cluster Role for Ansible

An Ansible Role for setting up Consul (https://consul.io)

## Organising hosts

This module requires 3 hosts groups: `consul`, `consul-server` and `consul-client`:

* All Consul servers should belong under the `consul` group.
* The servers should belong under `consul-server` group - Autojoin with Raft will bootstrap the cluster
* Any clients should belong under `consul-client` group

This role assumes a cluster of 5 nodes by default

### Example Hosts file

```
[consul:children]
consul-server
consul-client

[consul-server]
consul1.example.com
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
    bootstrap_expect: 3
    bind_adapter: ansible_eth0
    client_addr: 0.0.0.0
    datacenter: vagrant
    log_level: INFO
    data_dir: /var/lib/consul.d/
    encryption: <MY_ENCRYPTION_KEY>
  agents:
    - 172.16.10.22
    - 172.16.10.23
    - 172.16.10.24
    - 172.16.10.25
    - 172.16.10.26
```

This will create a 5 node cluster - the web UI will be available on any server node address.

## Generating Encryption Keys

The data and communication across the gossip protocol is encrypted. All nodes must use the same encryption key which is a random 16 byte string. You can generate this will the `consul keygen` command or the following if `consul` isn't available on your machine:

`dd if=/dev/urandom of=/dev/fd/1 bs=1 count=16 | base64`

Add the resulting key to your variables prior to running the role.

## Enable Cluster ACL

ACL for the cluster can be enabled by setting an ACL Master Token. If this isn't set ACL configuration is skipped. ACL is **not enabled by default**

```
consul:
  acl:
    acl_datacenter: dc1
    acl_default_policy: deny
    acl_down_policy: extend-cache
    acl_master_token: <MY_TOKEN>
```

When ACL is enabled a few default policies are applied to allow client syncing, DNS and anonymous consul member listing. Additional policies can be passed as json items under `acl.policies`. See [https://www.consul.io/docs/guides/acl.html](https://www.consul.io/docs/guides/acl.html) for ACL HTTP polcies

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
