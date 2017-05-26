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
	--extra-vars "overcloud_internal_api_fqdn=overcloud.internalapi.localdomain" \
	deploy-ipsec-tripleo.yml
```
