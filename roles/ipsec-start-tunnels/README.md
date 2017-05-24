Role Name
=========

Parses the given configuration file and starts the IPSEC tunnels.


Role Variables
--------------

* `ipsec_conf_file`: defines the file where the configuration will be read to get the names of the tunnels. Defaults to: /etc/ipsec.d/overcloud-tunnels.conf

Dependencies
------------

While there is no hard-dependency on other roles. Most likely the overcloud-ipsec-setup role will be ran, since that writes out the configuration.

Example Playbook
----------------

    - hosts: servers
      roles:
         - ipsec-setup
         - overcloud-ipsec-setup
         - ipsec-start-tunnels

