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

**Encryption algorithm**

It's possible to configure the encryption algorithm and key size with the
`ipsec_algorithm` variable. This will set the `phase2alg` option in the
ipsec tunnels configurations. The default value is 'aes_gcm128-null' and
the possible values should be checked in libreswan's documentation.
