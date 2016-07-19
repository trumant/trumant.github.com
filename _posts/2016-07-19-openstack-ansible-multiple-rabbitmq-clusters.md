---
layout: post
title:  "Multiple RabbitMQ clusters with OpenStack-Ansible"
categories:
  - openstack
  - openstack-ansible
  - rabbitmq
  - ceilometer
permalink: "/openstack-ansible-multiple-rabbitmq-clusters.html"
---
OpenStack-Ansible can automate the installation, configuration and upgrade process of a RabbitMQ cluster using built-in inventory groups, playbooks and roles. By default, it expects to manage only a single RabbitMQ cluster.

However a deployer can extend the functionality via configuration and use OpenStack-Ansible to manage multiple clusters.

## Architecture

OpenStack uses messaging infrastructure for RPC and event notification purposes. In large installations, it may be desirable to separate these types of traffic.

In the example architecture presented here, one RabbitMQ cluster is used for all RPC messaging, and a separate cluster is used for service notifications and telemetry events.

## Service RPC cluster

OpenStack-Ansible requires the following example configuration in order to manage the RPC cluster.

### `/etc/openstack_deploy/openstack_user_config.yml`

```yaml
# These hosts will provide the physical infrastructure for
# LXC containers that will run a Galera cluster, a RabbitMQ cluster
# and memcached services
shared-infra_hosts:
  infra1.oslab:
    ip: 192.168.1.20
  infra2.oslab:
    ip: 192.168.1.21
  infra3.oslab:
    ip: 192.168.1.22
```

When OpenStack-Ansible's dynamic inventory system processes the configuration above, it will generate inventory groups for our Rabbit MQ RPC cluster.

| Group | Description |
| --- | --- |
| shared-infra_hosts | Physical hosts that will run the LXC containers |
| rabbitmq | Contains 3 LXC container hosts that will run RabbitMQ |
| rabbitmq_all | Umbrella group containing the rabbitmq group |


## Notification and events cluster

To add a second cluster, we take advantage of the flexibility of OpenStack-Ansible inventory configuration and create a new configuration file for our new inventory groups that will be used for the second cluster.

### `/etc/openstack_deploy/env.d/telemetry_rabbitmq.yml`

```yaml
component_skel:
  telemetry_rabbitmq:
    belongs_to:
      - telemetry_rabbitmq_all

container_skel:
  telemetry_rabbitmq_container:
    belongs_to:
      # This is a group of containers mapped to a physical host.
      - telemetry-infra_containers
    contains:
      # This maps back to an item in the component_skel.
      - telemetry_rabbitmq
    properties:

physical_skel:
  # This maps back to items in the container_skel.
  telemetry-infra_containers:
    belongs_to:
      - all_containers
  # This is a required pair for the container physical entry.
  telemetry-infra_hosts:
    belongs_to:
      - hosts
```

When OpenStack-Ansible's dynamic inventory system processes the configuration above, it will generate inventory groups for our telemetry messaging cluster.

| Group | Description |
| --- | --- |
| telemetry-infra_hosts | Physical hosts that will run the LXC containers |
| telemetry_rabbitmq | Contains 3 LXC container hosts that will run RabbitMQ |
| telemetry_rabbitmq_all | Umbrella group containing the telemetry_rabbitmq group |

## Build the clusters

```shell
cd /opt/openstack-ansible/playbooks
openstack-ansible setup-hosts.yml
openstack-ansible rabbitmq-install.yml
openstack-ansible -e "rabbitmq_host_group=telemetry_rabbitmq_all" rabbitmq-install.yml
```