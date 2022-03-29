## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Network Diagram](https://github.com/rrazumov-rrs/rrazumov-rrs/blob/main/Diagrams/ELK_STACK_PROJECT-ORIGINAL.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _____ file may be used to install only certain pieces of it, such as Filebeat.

_![ansible config file](https://github.com/rrazumov-rrs/rrazumov-rrs/blob/main/)_
_![ansible hosts file](https://github.com/rrazumov-rrs/rrazumov-rrs/blob/main/)_

_![ansible dvwa setup playbook](https://github.com/rrazumov-rrs/rrazumov-rrs/blob/main/)_
_![ansible elk setup playbook](https://github.com/rrazumov-rrs/rrazumov-rrs/blob/main/)_
_![ansible filebeat setup playbook](https://github.com/rrazumov-rrs/rrazumov-rrs/blob/main/)_
_![ansible metricbeat setup playbook](https://github.com/rrazumov-rrs/rrazumov-rrs/blob/main/)_

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D__n Vulnerable Web Application.

Load balancing ensures that the application will be highly _____, in addition to restricting _____ to the network.
- _TODO: What aspect of security do load balancers protect? What is the advantage of a jump box?_

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _____ and system _____.
- _TODO: What does Filebeat watch for?_
- _TODO: What does Metricbeat record?_

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

|  **NAME**  |   **FUNCTION**   |            **IP ADDRESS**           | **EXPOSED PORTS** | **OPERATING SYSTEM** |      **DOCKER CONTAINERS**      |            **ALLOWED IP**           |                                      **DESCRIPTION**                                     |
|:----------:|:----------------:|:-----------------------------------:|:-----------------:|----------------------|:-------------------------------:|:-----------------------------------:|:----------------------------------------------------------------------------------------:|
| _JUMP BOX_ |    SSH GATEWAY   | PUB: 20.70.26.124<br>PRIV: 10.0.0.4 |         22        |  UBUNTU 20.04 GEN-2  |  ANSIBLE, FILEBEAT, METRICBEAT  | 108.168.111.241 SSH<br>10.0.0.4 SSH | ONLY MACHINE USED FOR SSH FROM EXTERNAL NETWORK<br>ANSIBLE USED TO SSH TO OTHER MACHINES |
|  _DVWA-1_  |  DVWA WEB SERVER |               10.0.2.4              |         80        |  UBUNTU 20.04 GEN-2  |    DVWA, FILEBEAT, METRICBEAT   |             10.0.0.4 SSH            |                                                                                          |
|  _DVWA-2_  |  DVWA WEB SERVER |               10.0.2.5              |         80        |  UBUNTU 20.04 GEN-2  |    DVWA, FILEBEAT, METRICBEAT   |             10.0.0.4 SSH            |                                                                                          |
|  _DVWA-3_  |  DVWA WEB SERVER |               10.0.2.6              |         80        |  UBUNTU 20.04 GEN-2  |    DVWA, FILEBEAT, METRICBEAT   |             10.0.0.4 SSH            |                                                                                          |
|   _ELK-1_  | ELK STACK SERVER |              10.10.1.4              |   5601,9200,5044  |  UBUNTU 20.04 GEN-2  | ELK STACK, FILEBEAT, METRICBEAT |             10.0.0.4 SSH            |                                                                                          |
|   _ELK-2_  | ELK STACK SERVER |              10.10.1.5              |   5601,9200,5044  |  UBUNTU 20.04 GEN-2  | ELK STACK, FILEBEAT, METRICBEAT |             10.0.0.4 SSH            |                                                                                          |
|  _LB-DVWA_ |  LOAD BALANCING  |         PUB: 20.213.234.237         |         80        |           -          |          HEALTH PROBES          |         108.168.111.241 HTTP        |                                                                                          |
|  _LB-ELK_  |  LOAD BALANCING  |          PUB: 13.77.60.190          |      5601,80      |           -          |          HEALTH PROBES          |         108.168.111.241 HTTP        |                                                                                          |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the _____ machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _TODO: Add whitelisted IP addresses_

Machines within the network can only be accessed by _____.
- _TODO: Which machine did you allow to access your ELK VM? What was its IP address?_

A summary of the access policies in place can be found in the table below.

| **PRIORITY** | **PORT** | **PROTOCOL** | **SOURCE** | **DESTINATION** | **ACTION** | **DESCRIPTION** |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| _4070_ | 80 | TCP | AzureLoadBalancer | VirtualNetwork | ALLOW | LOADBALLANCER HEALTH PROBES |
| _4071_ | 5601,9200 | TCP | VirtualNetwork | VirtualNetwork | ALLOW | SEND STATISTICS TO ELK |
| _4072_ | 22 | TCP | 10.0.0.4 | 10.0.1.0/24 | ALLOW | SSH FROM JBOX TO DVWA |
| _4080_ | 22 | TCP | 108.168.111.241 | 10.0.0.4 | ALLOW | SSH FROM HOME TO JBOX |
| _4081_ | 80 | TCP | 108.168.111.241 | 10.0.1.0/24 | ALLOW | HTTP FROM HOME TO DVWA |
| _4096_ | ANY | ANY | ANY | ANY | DENY | DENY ALL TRAFFIC ON VNET |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _TODO: What is the main advantage of automating configuration with Ansible?_

The playbook implements the following tasks:
- _TODO: In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
- ...
- ...

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- _TODO: List the IP addresses of the machines you are monitoring_

We have installed the following Beats on these machines:
- _TODO: Specify which Beats you successfully installed_

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the _____ file to _____.
- Update the _____ file to include...
- Run the playbook, and navigate to ____ to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

### Optional configurations


The files in this repository were used to configure the network depicted below.

![Network Diagram](https://github.com/rrazumov-rrs/rrazumov-rrs/blob/main/Diagrams/ELK_STACK_PROJECT-BONUS.png)

Changes made:

- Added another elk slack server for redundancy
- Added both servers on the load balancer with public ip address
- Load balancer automatically redirects requests on port 80 to port 5601
- Both DVWA and ELK are accessible not only by ip address, but by FQDN

**Trying to add shorter and more custom FQDN**