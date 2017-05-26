Role Name
=========

This sets up packages and firewall settings.

Sets the configuration for the IPSEC tunnels in the overcloud nodes.

Parses the given configuration file and starts the IPSEC tunnels.

Role Variables
--------------

* `overcloud_internal_api_fqdn`: The FQDN that's used for the overcloud's internal API VIP interface. Defaults to: 'overcloud.internalapi.localdomain'
* `ipsec_conf_file`: defines the file where the configuration will be written. Defaults to: /etc/ipsec.d/overcloud-tunnels.conf

Dependencies
------------

Requires that the ipsec-setup role has been ran first.

Example Playbook
----------------

    - hosts: servers
      roles:
         - overcloud-ipsec-setup
