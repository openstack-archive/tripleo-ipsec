Role Name
=========

This sets up packages and firewall settings.

Sets the configuration for the IPSEC tunnels in the overcloud nodes.

Parses the given configuration file and starts the IPSEC tunnels.

In a final step, when pacemaker is enabled, it enables resource agents for each
Virtual IP which puts up/tears down IPSEC tunnels depending on the VIP
location.

Role Variables
--------------

* `ipsec_psk`: the Pre-Shared Key to be used for the IPSEC tunnels.
  Note that is is sensible information and it's recommended that it's stored
  securely on the host where the playbook runs from, e.g. using Ansible Vault.
  One can generate this variable with the following command:
  `openssl rand -base64 48`
* `overcloud_controller_identifier`: This identifies which nodes are
  controllers in the cluster and which aren't, and should be part of the
  hostname of the controller. Defaults to: 'controller'. It's highly
  recommended that there's a way to explicitly identify the nodes this way.
* `ipsec_algorithm`: Defines the encryption algorithm to use in the phase2alg
  configuration option for the tunnels. Defaults to: `aes_gcm128-null`.
  The possible values should be checked in libreswan's documentation.
* `ipsec_skip_firewall_rules`: Determines whether the role should skip
  or not the firewall rules. Defaults to: `false`.

Example Playbook
----------------

    - hosts: servers
      roles:
         - tripleo-ipsec

Enabling ipsec tunnels in TripleO
=========================================

The main playbook to be ran on the overcloud nodes is:

```
tests/deploy-ipsec-tripleo.yml
```

Which will deploy IPSEC on the overcloud nodes for the internal API network.

We'll use a PSK and an AES128 cipher.

Add the PSK to an ansible var file:

```
cat <<EOF > ipsec-psk.yml
ipsec_psk: $(openssl rand -base64 48)
EOF
```

Note that for convenience I put the file in a path that's reachable for
ansible. And this name is necessary, as it's written directly to the playbook.

Encrypt the file with ansible-vault (note that it'll prompt for a password):

```
ansible-vault encrypt ipsec-psk.yml
```

Having done this, now you can run the playbook:

```
ansible-playbook -i /usr/bin/tripleo-ansible-inventory --ask-vault-pass \
	tests/deploy-ipsec-tripleo.yml
```

Generating an inventory
-----------------------

The script _/usr/bin/tripleo-ansible-inventory_ generates a dynamic inventory
with the nodes in the overcloud. And However it comes with some inconveniences:

* In deployments older than Pike, it might be a bit slow to run. To address
  this, in Ocata and Pike it's possible to generate a static inventory out of
  the output of this command:

  ```
  /usr/bin/tripleo-ansible-inventory  --static-inventory nodes.txt
  ```

  This will create a called nodes.txt with the static inventory, which we could
  now use and save some time.

* Newton unfortunately only takes into account computes and controllers with
  this command. So for this deployment we need to generate an inventory of our
  own. we can do so with the following command:

  ```
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
  ```

  This assumes that you're deploying this playbook from the undercloud itself.
  Hence the undercloud group containing localhost.
