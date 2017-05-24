Role Name
=========

Sets the configuration for the IPSEC tunnels in the overcloud nodes.

Role Variables
--------------

* `ipsec_conf_file`: defines the file where the configuration will be written. Defaults to: /etc/ipsec.d/overcloud-tunnels.conf

Dependencies
------------

Requires that the ipsec-setup role has been ran first.

Example Playbook
----------------

    - hosts: servers
      roles:
         - ipsec-setup
         - overcloud-ipsec-setup
