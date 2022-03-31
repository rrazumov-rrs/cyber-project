This is the configuration files for the ansible container. Config file defines the default user with sudo priveleges that will be executing playbooks on machines.

The _EXTRA_ step will be to add the list of all of the machines on the network and use this host for some of the playbooks (ie. upgrading systems) to further automate some processes.


/etc/ansible/ansible.cfg

line - 107

![Remote user](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/ANSIBLE-REMOTE-USER.png)


/etc/ansible/hosts

line - 107

![Remote user](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/ANSIBLE-HOSTS-FILE.png)