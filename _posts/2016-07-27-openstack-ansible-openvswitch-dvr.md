---
layout: post
title:  "Configuring OpenStack-Ansible for Distributed Virtual Routing with Open vSwitch"
categories:
  - openstack
  - openstack-ansible
  - neutron
  - openvswitch
permalink: "/openstack-ansible-dvr-with-openvswitch.html"
---
Following up from a prior post, [Configuring OpenStack-Ansible for Open vSwitch]({% post_url 2016-07-05-openstack-ansible-openvswitch %}), the [OpenStack-Ansible project](http://governance.openstack.org/reference/projects/openstackansible.html) has recently added support for enabling Distributed Virtual Routing when using the Open vSwitch ML2 Neutron agent. Support landed in the master branch with [these patches](https://review.openstack.org/#/q/status:merged+topic:bp/neutron-dvr) and will be released in the Newton cycle.

Now, deployers who wish to use Open vSwitch can also take advantage of the high availability benefits of DVR. With this change, the OpenStack-Ansible project now supports the [Scenario: High Availability using Distributed Virtual Routing](http://docs.openstack.org/mitaka/networking-guide/scenario-dvr-ovs.html) described in the [OpenStack Networking guide](http://docs.openstack.org/mitaka/networking-guide/).

## Sample Configuration

Using the same lab hardware and network architecture I described in my [previous post]({% post_url 2016-07-05-openstack-ansible-openvswitch %}) and running the master version of OpenStack-Ansible, my configuration for enabling DVR looks like:

### /etc/openstack_deploy/user_variables.yml

```yaml
# Ensure the openvswitch kernel module is loaded
openstack_host_specific_kernel_modules:
  - name: "openvswitch"
    pattern: "CONFIG_OPENVSWITCH="
    group: "network_hosts"

### Neutron specific config
neutron_plugin_type: ml2.ovs.dvr

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

You may notice that only a single line has changed in this configuration. Where previously, I had `neutron_plugin_type: ml2.ovs`, I now have `neutron_plugin_type: ml2.ovs.dvr`

With that simple change, OpenStack-Ansible now knows to install the Neutron L3 and metadata agents on the compute hosts and is able to configure the L3 agent's `agent_mode` appropriately depending on whether or not the agent is running on a network or compute host.

## Conclusion

If you've been waiting for DVR support in OpenStack-Ansible, please give it a try and be sure to provide any feedback to the community. Bug reports in [LaunchPad](https://launchpad.net/openstack-ansible), questions in IRC at [#openstack-ansible on FreeNode](http://webchat.freenode.net/?channels=openstack-ansible) and emails to the [OpenStack-Dev list](http://lists.openstack.org/cgi-bin/mailman/listinfo/openstack-dev) are all great ways to reach out.
