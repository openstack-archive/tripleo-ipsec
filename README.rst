tripleo-ipsec
=============

Ansible role to configure IPSEC tunnels for TripleO

* This sets up packages and firewall settings.

* Sets the configuration for the IPSEC tunnels in the overcloud nodes.

* Parses the given configuration file and starts the IPSEC tunnels.

In a final step, when pacemaker is enabled, it enables resource agents for each
Virtual IP which puts up/tears down IPSEC tunnels depending on the VIP
location.

Note that as of the latest code, this now relies on the usage of TripleO's
dynamic inventory. This means that it expects the inventory to tell the role
which networks are being set and which IPs do the hosts have. If the relevant
variables don't come from the inventory, the role will attempt to use the legacy
setup which autodiscovers these. However, this setup is not very reliable if
you're using custom networks.

Role Variables
--------------

* `ipsec_psk`: the Pre-Shared Key to be used for the IPSEC tunnels.
  Note that is is sensible information and it's recommended that it's stored
  securely on the host where the playbook runs from, e.g. using Ansible Vault.
  One can generate this variable with the following command:
  `openssl rand -base64 48`
* `ipsec_algorithm`: Defines the encryption algorithm to use in the phase2alg
  configuration option for the tunnels. Defaults to: `aes_gcm128-null`.
  The possible values should be checked in libreswan's documentation.
* `ipsec_configure_vips`: Determines whether or not the role should configure
  the tunnels for the VIPs. Defaults to: `true`.
* `ipsec_skip_firewall_rules`: Determines whether the role should skip
  or not the firewall rules. Defaults to: `false`.
* `ipsec_uninstall_tunnels`: Determines whether the role should remove the IPSEC
  tunnels that were previously set. Defaults to: `false`.
* `ipsec_upgrade_tunnels`: Determines whether the role should upgrade the IPSEC
  tunnels that were previously set. This means it'll remove all the tunnels
  created in a previous run and replace them. Defaults to: `false`.
* `ipsec_setup_resource_agents`: Determines whether the role should create the
  pacemaker resource agents or not. Defaults to: `true`.
* `ipsec_skip_networks`: Determines which networks should be skipped. defaults to `[]`.
* `ipsec_force_install_legacy`: Forces the legacy installation. Defaults to: `false`.
* `overcloud_controller_identifier`: This identifies which nodes are
  controllers in the cluster and which aren't, and should be part of the
  hostname of the controller. Defaults to: 'controller'. It's highly
  recommended that there's a way to explicitly identify the nodes this way.
  Note that this is only used in the legacy setup.

Example Playbook
----------------

Sample::

   - hosts: servers
     roles:
        - tripleo-ipsec

Enabling ipsec tunnels in TripleO
=================================

The main playbook to be ran on the overcloud nodes is::

   tests/deploy-ipsec-tripleo.yml

Which will deploy IPSEC on the overcloud nodes for the internal API network.

We'll use a PSK and an AES128 cipher.

Add the PSK to an ansible var file::

   cat <<EOF > ipsec-psk.yml
   ipsec_psk: $(openssl rand -base64 48)
   EOF

Encrypt the file with ansible-vault (note that it'll prompt for a password):

   ansible-vault encrypt ipsec-psk.yml

Having done this, now you can run the playbook::

   ansible-playbook -i /usr/bin/tripleo-ansible-inventory --ask-vault-pass \
           -e @ipsec-psk.yml tests/deploy-ipsec-tripleo.yml

Generating an inventory
-----------------------

The script */usr/bin/tripleo-ansible-inventory* generates a dynamic inventory
with the nodes in the overcloud. And However it comes with some inconveniences:

* In deployments older than Pike, it might be a bit slow to run. To address
  this, in Ocata and Pike it's possible to generate a static inventory out of
  the output of this command::

     /usr/bin/tripleo-ansible-inventory  --static-inventory nodes.txt

  This will create a called nodes.txt with the static inventory, which we could
  now use and save some time.

* Newton unfortunately only takes into account computes and controllers with
  this command. So for this deployment we need to generate an inventory of our
  own. we can do so with the following command::

     cat <<EOF > nodes.txt
     [undercloud]
     localhost

     [undercloud:vars]
     ansible_connection = local

     [overcloud:vars]
     ansible_ssh_user = heat-admin

     [overcloud]
     $( openstack server list -c Networks -f value | sed 's/ctlplane=//')
     EOF

  This assumes that you're deploying this playbook from the undercloud itself.
  Hence the undercloud group containing localhost.

Skipping networks
=================

The `ipsec_skip_networks` variable allows the user to skip the tunnel setup
for certain networks. This works by using the network name, which can vary
depending on your type of setup.

Using the dynamic inventory (Queens and beyond)
-----------------------------------------------

When using the dynamic inventory, the network names will be based on the names
that are set in your `network_data.yaml` file, from tripleo-heat-templates.
As mentioned in tripleo-heat-templates, this file will determine which networks
you're setting up in your overall TripleO deployment, and will even specify
which of those networks have VIPs attached to them.

The network names to use in the `ipsec_skip_networks` variable will be under
the `name_lower` section of each network definition.

For instance, if you want to skip the storage management network, you'll see
that the entry looks as follows::

  - name: StorageMgmt
    name_lower: storage_mgmt
    vip: true
    vlan: 40
    ip_subnet: '172.16.3.0/24'
    allocation_pools: [{'start': '172.16.3.4', 'end': '172.16.3.250'}]
    ipv6_subnet: 'fd00:fd00:fd00:4000::/64'
    ipv6_allocation_pools: [{'start': 'fd00:fd00:fd00:4000::10', 'end': 'fd00:fd00:fd00:4000:ffff:ffff:ffff:fffe'}]

So, in this case, the variable you'll put in your ansible variables file will
have the following entry::

  ipsec_skip_networks:
  - storage_mgmt

You can add more networks by adding more items to that list.

Legacy setups
-------------

If you're using a legacy setup (which would work in Newton), you'll need to
note that the network names are hardcoded; so you'll have the following
options available:

* internalapi
* storage
* storagemgmt
* ctlplane

You can also explicitly skip creating the Redis VIP by adding the `redis` word
to the list.

If you would want to skip the Storage and Storage Management networks, the
variable you'll put in your ansible variables file will have the
following entry::

  ipsec_skip_networks:
  - storage
  - storagemgmt
