POC for enabling ipsec tunnels in TripleO
=========================================

The main playbook to be ran on the overcloud nodes is:

```
deploy-ipsec-tripleo.yml
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
	deploy-ipsec-tripleo.yml
```

Extra configuration options
---------------------------

**Internal API VIP FQDN**

For releases newer than Newton, it is possible to configure FQDNs for the
internal network VIPs. This is controlled by the CloudName<network> variables.
If this is set, you need to add this configuration to the playbook by setting
the `overcloud_internal_api_fqdn` variable in the following manner:

```
ansible-playbook -i /usr/bin/tripleo-ansible-inventory --ask-vault-pass \
	--extra-vars "overcloud_internal_api_fqdn=overcloud.internalapi.mydomain" \
	deploy-ipsec-tripleo.yml
```

**Controller identifier**

For simplicity, this playbook uses a string identifier to know which nodes in
the cluster are controllers and which aren't. It takes this from the hostnames.
So it's highly recommended that there's a way to explicitly identify the nodes
this way. To set this, you need to modify the `overcloud_controller_identifier`
with an identifier of your choice.

**Encryption algorithm**

It's possible to configure the encryption algorithm and key size with the
`ipsec_algorithm` variable. This will set the `phase2alg` option in the
ipsec tunnels configurations. The default value is 'aes_gcm128-null' and
the possible values should be checked in libreswan's documentation.

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
  [overcloud:vars]
  ansible_ssh_user = heat-admin

  [overcloud]
  $( openstack server list -c Networks -f value | sed 's/ctlplane=//')
  EOF
  ```
