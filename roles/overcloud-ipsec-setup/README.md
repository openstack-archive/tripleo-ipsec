Role Name
=========

This sets up packages and firewall settings.

Sets the configuration for the IPSEC tunnels in the overcloud nodes.

Parses the given configuration file and starts the IPSEC tunnels.

Role Variables
--------------

* `overcloud_controller_identifier`: This identifies which nodes are
  controllers in the cluster and which aren't, and should be part of the
  hostname of the controller. Defaults to: 'controller'
* `ipsec_conf_file`: defines the file where the configuration will be written.
  Defaults to: /etc/ipsec.d/overcloud-tunnels.conf
* `ipsec_algorithm`: Defines the encryption algorithm to use in the phase2alg
  configuration option for the tunnels. Defaults to: 'aes_gcm128-null'

Dependencies
------------

Requires that the ipsec-setup role has been ran first.

Example Playbook
----------------

    - hosts: servers
      roles:
         - overcloud-ipsec-setup
