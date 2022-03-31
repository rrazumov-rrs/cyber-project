## ELK Stack Deployment

This repository contains the following details:
- Description of the Topology
- Network Configuration
- VM configuration
- Access Policies
- DVWA Configuration
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The files in this repository were used to configure the network depicted below. The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D__n Vulnerable Web Application.

The following diagram depicts the basic network configuration:
![Network Diagram](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/NET-DIAGRAM-ORIGINAL.png)


This diagram provides the overview of more advanced configuration. These will be discussed under __EXTRAS__ steps for each section.
![Network Diagram](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/NET-DIAGRAM-EXTRA.png)


### General Configuration

For this project we will be creating Azure Resource Group and adding the following resources to it:

**Basic setup:**
- Two(2) virtual networks
- Two(2) network security groups
- Four(4) virtual machines
  - One(1) jump box
  - Two(2) dvwa servers
  - One(1) elk server
- One(1) load balancer
- Three(3) public ip addresses
- One(1) availability set

**Extra setup:**
- Two(2) virtual networks
- Two(2) network security groups
- Six(6) virtual machines
  - One(1) jump box
  - Three(3) dvwa servers
  - Two(2) elk server
- Two(2) load balancer
- Three(3) public ip addresses
- Two(2) availability set
- Two(2) DNS records

Note that elk network and dvwa network are created in different regions and in both setup cases would need to have peer connection enabled.

![Peer Connection](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VNET-PEERING.png)

### VM Configuration

For this project, the key-based ssh authentication is used and first step is to generate ssh keypair:

![Ssh Keygen](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/SSH-KEYGEN.png)

The are three options for the VM sizes that we are using on the network.

![Vm Size](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VM-SIZES.png)

The sizes can be any of the available in the region, however the following are the minimum requirements for each of the machines:
- JUMP BOX: Standard_B1s
- DVWA SERVERS: Standard_B1ms
- ELK SERVERS: Standard_B2s

Since only **JUMP BOX** is allowed to have ssh connection to other machines on the network and the only one to have the publicly accessible ssh connection, this machine will be configured before moving to other machine configuration. Use the ssh public key as the user authentication method.

![Ssh key](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VM-SSH-KEY.png)

The rest of the machines are being configured from the ansible container that is installed on the jbox. The public key used to ssh on other virtual machines is generated on the jbox inside the ansible container.


Docker installation setup:
- apt -y update
- apt -y install docker.io
- systemctl enable docker
- systemctl start docker
- docker pull cyberxsecurity/ansible

Ansible basic setup:
- docker run -it cybersecurity/ansible:latest bash
- docker start magical_albatani
- docker attach magical_albatani

For the extra steps, the ansible container can be given a name in place the random name as well as start the container automatically on system restart.

Ansible extra setup:
- docker run -it --name jbox --restart always cyberxsecurity/ansible:latest bash
- docker attach jbox


### Access Policies

Next the Network Security Group is configured to control who can access the network resources. The DVWA servers are only allowed to be accessed by the 'home' network that is designated for pentesting.

The following NSG rules are the basic security configuration that needed:

| **PRIORITY** | **PORT** | **PROTOCOL** |     **SOURCE**    | **DESTINATION** | **ACTION** |             **DESCRIPTION**            |
|:------------:|:--------:|:------------:|:-----------------:|:---------------:|:----------:|:--------------------------------------:|
|    _3990_    |    80    |      TCP     |  108.168.111.241  |   10.0.2.0/24   |    ALLOW   | ALLOW CONNECTION TO THE WEB SERVERS    |
|    _3991_    |    80    |      TCP     | AzureLoadBalancer |  VirtualNetwork |    ALLOW   | ALLOW HEALTH PROBES FOR WEB SERVERS    |
|    _4000_    |    22    |      TCP     |        ANY        |     10.0.0.4    |    ALLOW   | ALLOW SSH CONNECTION TO JBOX           |
|    _4001_    |    22    |      TCP     |      10.0.0.4     |  VirtualNetwork |    ALLOW   | ALLOW SSH CONNECTION TO OTHER MACHINES |

*EXTRA*

To further restrict the traffic from external and internal sources we will be modifying the NSG to include additional rule as well as modifying existing one. Here is the resulting NSG for DVWA network:

| **PRIORITY** | **PORT** | **PROTOCOL** |     **SOURCE**    | **DESTINATION** | **ACTION** |             **DESCRIPTION**            |
|:------------:|:--------:|:------------:|:-----------------:|:---------------:|:----------:|:--------------------------------------:|
|    _3990_    |    80    |      TCP     |  108.168.111.241  |   10.0.2.0/24   |    ALLOW   | ALLOW CONNECTION TO THE WEB SERVERS    |
|    _3991_    |    80    |      TCP     | AzureLoadBalancer |  VirtualNetwork |    ALLOW   | ALLOW HEALTH PROBES FOR WEB SERVERS    |
|    _4000_    |    22    |      TCP     |  108.168.111.241  |     10.0.0.4    |    ALLOW   | ALLOW SSH CONNECTION TO JBOX           |
|    _4001_    |    22    |      TCP     |      10.0.0.4     |  VirtualNetwork |    ALLOW   | ALLOW SSH CONNECTION TO OTHER MACHINES |
|    _4096_    |    ANY   |      ANY     |        ANY        |       ANY       |    DENY    | DENY ALL TRAFFIC ON VNET               |





### JBOX Configuration
### DVWA Configuration
### ELK Configuration



These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the config and playbook file may be used to install only certain pieces of it, such as Filebeat.

Load balancing ensures that the DVWA web server will be highly available, in addition network security groups will be restricting access to the network.

Since the DVWA servers are using load balancing, the access to the servers will be through the load balancer's public IP address.
Ssh only allowed to 



[Ansible](https://github.com/rrazumov-rrs/cyber-project/tree/main/ANSIBLE) configuration files:
- _[ansible config file](https://github.com/rrazumov-rrs/cyber-project/blob/main/ANSIBLE/ansible.cfg)_
- _[ansible hosts file](https://github.com/rrazumov-rrs/cyber-project/blob/main/ANSIBLE/hosts)_

[Beats](https://github.com/rrazumov-rrs/cyber-project/blob/main/BEATS) configuration files:
- _[filebeat config file](https://github.com/rrazumov-rrs/cyber-project/blob/main/BEATS/filebeat-config.yml)_
- _[metsicbeat config file](https://github.com/rrazumov-rrs/cyber-project/blob/main/BEATS/metricbeat-config.yml)_

[Ansible](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS) playbooks:
- _[dvwa setup playbook](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS)_
- _[elk setup playbook](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS)_
- _[filebeat setup playbook](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS)_
- _[metricbeat setup playbook](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS)_
- _[upgrade systems playbook](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS)_











Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
- _TODO: What aspect of security do load balancers protect? What is the advantage of a jump box?_

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _____ and system _____.
- _TODO: What does Filebeat watch for?_
- _TODO: What does Metricbeat record?_

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

|  **NAME**  |         **IP ADDRESS**         |        **EXPOSED PORTS**       |      **DOCKER CONTAINERS**      |
|:----------:|:------------------------------:|:------------------------------:|:-------------------------------:|
| _JUMP BOX_ | PUB:20.70.26.124 PRIV:10.0.0.4 |    22 (108.168.111.241 ONLY)   |  ANSIBLE, FILEBEAT, METRICBEAT  |
|  _DVWA-1_  |    PUB:LB-DVWA PRIV:10.0.2.4   |        80 (LB-DVWA ONLY)       |    DVWA, FILEBEAT, METRICBEAT   |
|  _DVWA-2_  |    PUB:LB-DVWA PRIV:10.0.2.5   |        80 (LB-DVWA ONLY)       |    DVWA, FILEBEAT, METRICBEAT   |
|  _DVWA-3_  |    PUB:LB-DVWA PRIV:10.0.2.6   |        80 (LB-DVWA ONLY)       |    DVWA, FILEBEAT, METRICBEAT   |
|   _ELK-1_  |    PUB:LB-ELK PRIV:10.10.1.4   |         5601,9200,5044         | ELK STACK, FILEBEAT, METRICBEAT |
|   _ELK-2_  |    PUB:LB-ELK PRIV:10.10.1.5   |         5601,9200,5044         | ELK STACK, FILEBEAT, METRICBEAT |
|  _LB-DVWA_ |       PUB:20.213.234.237       |    80 (108.168.111.241 ONLY)   |                                 |
|  _LB-ELK_  |        PUB:52.243.67.47        | 5601,80 (108.168.111.241 ONLY) |                                 |



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