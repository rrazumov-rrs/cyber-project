This is the configuration files for the ansible container. Config file defines the default user with sudo priveleges that will be executing playbooks on machines.

/etc/ansible/ansible.cfg

line - 107

![Remote user](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/ANSIBLE-REMOTE-USER.png)

The hosts file contains the list of different host groups that will be called by ansible playbook when runnig them. Host groups include the ip addresses of all of the machines that have specific function. 

**EXTRA** - added the 'test' host list of the ranges of machines on the network in the event that the more machines are added to the network. This can also be used to automate some tasks like upgrading the systems

/etc/ansible/hosts

line - 107

![Remote user](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/ANSIBLE-HOSTS-FILE.png)