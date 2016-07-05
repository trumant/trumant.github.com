---
layout: post
title:  "Configuring OpenStack-Ansible for Open vSwitch"
categories:
  - openstack
  - openstack-ansible
  - neutron
  - openvswitch
permalink: "/openstack-ansible-openvswitch.html"
---
The [OpenStack-Ansible project](http://governance.openstack.org/reference/projects/openstackansible.html) has recently added support for the Open vSwitch ML2 neutron agent in the Newton release cycle.

I was very excited to see this feature get merged and wanted to test it out in a home lab environment. My goal was to use OpenStack-Ansible to mimic the [Scenario: Classic with Open vSwitch neutron configuration](http://docs.openstack.org/mitaka/networking-guide/scenario-classic-ovs.html) described in the [OpenStack Networking guide](http://docs.openstack.org/mitaka/networking-guide/). The resulting OpenStack environment will have a single provider VLAN network and up to 98 VLAN tenant networks.

## Lab Hosts

To keep things super simple, I used a two host environment made up of a controller and infrastructure host: `man1.oslab` and compute host: `comp1.oslab`. Both hosts have two network interfaces: eth0 and eth1.

## Lab Network

The lab network architecture provides the following networks:

  * Management network - `192.168.1.0/24`
  * Provider network - `192.168.2.0/24` using VLAN tag 101
  * Project/Tenant networks - VLAN tags 102-199

In order to speed up the installation process somewhat, I've chosen to deploy only the following OpenStack services: keystone, glance, nova and neutron.

## Host Network Configuration

OpenStack-Ansible is going to expect 2 Linux bridges: `br-mgmt` and `br-vlan` on each host based on our configuration above. I've configured the hosts as follows:

### /etc/network/interfaces

```bash
auto lo
iface lo inet loopback

# Management network
auto eth0
iface eth0 inet manual

# VLAN network
auto eth1
iface eth1 inet manual

source /etc/network/interfaces.d/*.cfg
```

### /etc/network/interfaces.d/br-mgmt.cfg

```bash
# OpenStack Management network bridge
auto br-mgmt
iface br-mgmt inet static
  bridge_stp off
  bridge_waitport 0
  bridge_fd 0
  bridge_ports eth0
  address MANAGEMENT_NETWORK_IP
  netmask 255.255.255.0
  gateway 192.168.1.1
  dns-nameservers 8.8.8.8 8.8.4.4
```

###  /etc/network/interfaces.d/br-vlan.cfg

```bash
# OpenStack Networking VLAN bridge
auto br-vlan
iface br-vlan inet manual
  bridge_stp off
  bridge_waitport 0
  bridge_fd 0
  bridge_ports eth1
```

## Configuration

I define the network configuration and host groupings in

### /etc/openstack_deploy/openstack_user_config.yml

```yaml
---
cidr_networks:
  container: 192.168.1.0/24

used_ips:
  - "192.168.1.0,192.168.1.24" # lab metal hosts

global_overrides:
  internal_lb_vip_address: 192.168.1.20 # man1.oslab
  external_lb_vip_address: 192.168.1.20 # man1.oslab
  management_bridge: "br-mgmt"

  provider_networks:
    - network:
        group_binds:
          - all_containers
          - hosts
        type: "raw"
        container_bridge: "br-mgmt"
        container_interface: "eth1"
        container_type: "veth"
        ip_from_q: "container"
        is_container_address: true
        is_ssh_address: true
    - network:
        group_binds:
          - neutron_openvswitch_agent
        container_bridge: "br-vlan"
        container_interface: "eth12"
        container_type: "veth"
        type: "vlan"
        range: "102:199"
        net_name: "physnet1"

shared-infra_hosts:
  man1.oslab:
    ip: 192.168.1.20

repo-infra_hosts:
  man1.oslab:
    ip: 192.168.1.20

os-infra_hosts:
  man1.oslab:
    affinity:
      heat_apis_container: 0
      heat_engine_container: 0
    ip: 192.168.1.20

identity_hosts:
  man1.oslab:
    ip: 192.168.1.20

network_hosts:
  man1.oslab:
    ip: 192.168.1.20

compute_hosts:
  comp1.oslab:
    ip: 192.168.1.23

haproxy_hosts:
  man1.oslab:
    ip: 192.168.1.20

log_hosts:
  man1.oslab:
    ip: 192.168.1.20
```

And I define my user-specific overrides in

### /etc/openstack_deploy/user_variables.yml

```yaml
# Ensure the openvswitch kernel module is loaded
openstack_host_specific_kernel_modules:
  - name: "openvswitch"
    pattern: "CONFIG_OPENVSWITCH="
    group: "network_hosts"

### Neutron specific config
neutron_plugin_type: ml2.ovs

neutron_ml2_drivers_type: "flat,vlan"

# Typically this would be defined by the os-neutron-install
# playbook. The provider_networks library would parse the
# provider_networks list in openstack_user_config.yml and
# generate the values of network_types, network_vlan_ranges
# and network_mappings. network_mappings would have a
# different value for each host in the inventory based on
# whether or not the host was metal (typically a compute host)
# or a container (typically a neutron agent container)
#
# When using Open vSwitch, we override it to take into account
# the Open vSwitch bridge we are going to define outside of
# OpenStack-Ansible plays
neutron_provider_networks:
  network_flat_networks: "*"
  network_types: "vlan"
  network_vlan_ranges: "physnet1:102:199"
  network_mappings: "physnet1:br-provider"
```

## Installation

Run the OpenStack-Ansible playbooks to install and configure the two lab hosts

```
openstack-ansible setup-hosts.yml
openstack-ansible setup-infrastructure.yml
openstack-ansible os-keystone-install.yml
openstack-ansible os-glance-install.yml
openstack-ansible os-nova-install.yml
openstack-ansible os-neutron-install.yml
```

## Open vSwitch Configuration

I created a custom playbook for setting up the Open vSwitch bridges and ports that uses the OpenStack-Ansible inventory.

### /opt/openstack-ansible/playbooks/ovs-setup.yml

```yaml
---
- name: Setup OVS bridges
  hosts: neutron_openvswitch_agent
  user: root

  tasks:
    - name: Setup br-provider
      openvswitch_bridge:
        bridge: br-provider
        state: present
      notify:
        - Restart neutron-openvswitch-agent

    - name: Add port to br-provider
      openvswitch_port:
        bridge: br-provider
        port: "{{ properties.is_metal | default(false) | ternary('br-vlan','eth12') }}"
        state: present
      notify:
        - Restart neutron-openvswitch-agent

  handlers:
    - name: Restart neutron-openvswitch-agent
      service:
        name: neutron-openvswitch-agent
        state: restarted
```

I run that with:

```bash
cd /opt/openstack-ansible/playbooks
openstack-ansible ovs-setup.yml
```

## Deployment View

After installing OpenStack with OpenStack-Ansible plays and setting up Open vSwitch with the custom play, our deployment environment looks like:

![OpenStack-Ansible Open vSwitch Figure 1](/assets/osa_ovs.png)

## Create the networks

### Provider network

```bash
ssh man1.oslab
lxc-attach -n `lxc-ls | grep utility`
source openrc

# Create the provider network
neutron net-create physnet1 --shared \
  --provider:physical_network physnet1 \
  --provider:network_type vlan \
  --provider:segmentation_id 101

# Create the subnet for the provider net
neutron subnet-create physnet1 192.168.2.0/24 \
  --name physnet-subnet \
  --gateway 192.168.2.1
```

### Project/Tenant network

Requires a demo project to have been setup and a `demo-project-openrc` with the project credentials.

```bash
ssh man1.oslab
lxc-attach -n `lxc-ls | grep utility`
source demo-openrc

# Create the project network
neutron net-create project

# Create the subnet for the provider net
neutron subnet-create project 192.168.3.0/24 \
  --name project-subnet \
  --gateway 192.168.3.1
```

## Launch Instances

Requires flavor, image and key setup:

```bash
ssh man1.oslab
lxc-attach -n `lxc-ls | grep utility`
source demo-openrc

# Get the network ids
openstack network list

# Allow ping access
openstack security group rule create \
  --proto icmp default

# Launch an instance on the provider network
openstack server create \
  --flavor m1.nano \
  --image cirros \
  --security-group default \
  --nic net-id=UUID_OF_PROVIDER_NETWORK \
  --key-name mykey provider-instance

# Launch an instance on the project network
openstack server create \
  --flavor m1.nano \
  --image cirros \
  --security-group default \
  --nic net-id=UUID_OF_PROJECT_NETWORK \
  --key-name mykey project-instance
```

## Validate Instance Connectivity
```bash
ssh man1.oslab
lxc-attach -n `lxc-ls | grep neutron_agents_`

# ping provider instance from the provider network namespace
ip netns exec `ip netns | grep UUID_OF_PROVIDER_NETWORK` \
  ping -c 3 PROVIDER_INSTANCE_IP

# ping project instance from the project network namespace
ip netns exec `ip netns | grep UUID_OF_PROJECT_NETWORK` \
  ping -c 3 PROJECT_INSTANCE_IP
```

## Conclusion

With some basic configuration, I was able to get OpenStack-Ansible to install and configure a typical OpenStack neutron Open vSwitch scenario. While I have not yet played around with vxlan or gre tunneling, I suspect that it is as simple as making a few configuration tweaks.

To learn more about OpenStack-Ansible or to contribute to the community project, visit the project on [LaunchPad](https://launchpad.net/openstack-ansible) or join us on IRC in [#openstack-ansible on FreeNode](http://webchat.freenode.net/?channels=openstack-ansible)
