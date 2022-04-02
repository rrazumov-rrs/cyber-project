The *BEAT* an application that contains several modules that enable to collection of specific data from the target machine to the logstash server that is part of the ELK framework.

The **FILEBEAT** is the service that collects the logs on the target systems this allows to monitor and to analyze various activities like authentication attempts, file modification, and privilege escalation on the targeted systems.

The **METRISBEAT** on the other hand collects information about the "hardware" resources, and their availability such as CPU and RAM usage. 

This section contains two configuration files used to configure both __FILEBEAT__ and __METRICBEAT__ on the machines. In general, both files begin with the conigurations of different modules used for the beats and closer to the end of the configuration file is the section for the outputs. Output section will be edited to include the elk stack servers that the statistics will be sent to.

_*EXTRA*_ for multiple elk servers to be used on the network, all of their IP addresses have to be defined in configuration files.


/etc/ansible/dvwa-vm/filebeat-config.yml.

line 1096 - elasticsearch section.

![Network Diagram](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/FILEBEAT-ELASTIC.png)

line 1800 - kibana section.

![Network Diagram](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/FILEBEAT-KIBANA.png)


/etc/ansible/dvwa-vm/metricbeat-config.yml.

line 92 - elasticsearch section.

![Network Diagram](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/METRICBEAT-ELASTIC.png)

line 57 - kibana section.

![Network Diagram](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/METRICBEAT-KIBANA.png)
