## ELK Stack Deployment

This document contains the following details:
- Description of the Topology
- Project Requirements
- Access Policies
- VM configuration
- Ansible Configuration
- DVWA Configuration
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D__n Vulnerable Web Application.

These files have been tested and used to generate a live DVWA and ELK deployment on Azure. They can be used to either recreate the entire deployment pictured below. Alternatively, select portions of the config and playbook file may be used to install only certain pieces of it, such as Filebeat.

### Description of the Topology

The following diagram depicts the basic network configuration:
![Network Diagram](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/NET-DIAGRAM-ORIGINAL.png)


This diagram provides the overview of more advanced configuration. These will be discussed under **EXTRA** steps for each section.
![Network Diagram](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/NET-DIAGRAM-EXTRA.png)


### Project Requirements

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

**EXTRA setup:**
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


### Virtual Network Configuration

For this project, two virtual networks have been created in the same resource group, but different resions:

![Peer Connection](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VNET-CREATE.png)

The networks that are created in different regions are unable to communicate to oneanother and the new peering connection between the networks is created to transfer data between the networks:

![Peer Connection](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VNET-PEERING.png)

### Access Policies

Next, the networks need to be hardened by creating two Network Security Groups and configuring them to control who can access the network resources. The DVWA servers are only allowed to be accessed by the 'home' network that is designated for pentesting.

The following NSG rules are the basic security configuration that needed:

| **PRIORITY** | **PORT** | **PROTOCOL** |     **SOURCE**    | **DESTINATION** | **ACTION** |             **DESCRIPTION**            |
|:------------:|:--------:|:------------:|:-----------------:|:---------------:|:----------:|:--------------------------------------:|
|    _3990_    |    80    |      TCP     |  108.168.111.241  |   10.0.2.0/24   |    ALLOW   | ALLOW CONNECTION TO THE WEB SERVERS    |
|    _3991_    |    80    |      TCP     | AzureLoadBalancer |  VirtualNetwork |    ALLOW   | ALLOW HEALTH PROBES FOR WEB SERVERS    |
|    _4000_    |    22    |      TCP     |        ANY        |     10.0.0.4    |    ALLOW   | ALLOW SSH CONNECTION TO JBOX           |
|    _4001_    |    22    |      TCP     |      10.0.0.4     |  VirtualNetwork |    ALLOW   | ALLOW SSH CONNECTION TO OTHER MACHINES |


**EXTRA** - To further restrict the traffic from external and internal sources we will be modifying the NSG to include additional rule as well as modifying existing one. Here is the resulting NSG for DVWA network:

| **PRIORITY** | **PORT** | **PROTOCOL** |     **SOURCE**    | **DESTINATION** | **ACTION** |             **DESCRIPTION**            |
|:------------:|:--------:|:------------:|:-----------------:|:---------------:|:----------:|:--------------------------------------:|
|    _3990_    |    80    |      TCP     |  108.168.111.241  |   10.0.2.0/24   |    ALLOW   | ALLOW CONNECTION TO THE WEB SERVERS    |
|    _3991_    |    80    |      TCP     | AzureLoadBalancer |  VirtualNetwork |    ALLOW   | ALLOW HEALTH PROBES FOR WEB SERVERS    |
|    _4000_    |    22    |      TCP     |  108.168.111.241  |     10.0.0.4    |    ALLOW   | ALLOW SSH CONNECTION TO JBOX           |
|    _4001_    |    22    |      TCP     |      10.0.0.4     |  VirtualNetwork |    ALLOW   | ALLOW SSH CONNECTION TO OTHER MACHINES |
|    _4096_    |    ANY   |      ANY     |        ANY        |       ANY       |    DENY    | DENY ALL TRAFFIC ON VNET               |



When it comes to the Elk server NSG, the basic configuration would look like the following:

| **PRIORITY** |  **PORT** | **PROTOCOL** |     **SOURCE**    | **DESTINATION** | **ACTION** |             **DESCRIPTION**             |
|:------------:|:---------:|:------------:|:-----------------:|:---------------:|:----------:|:---------------------------------------:|
|    _3990_    |    5601   |      TCP     |        ANY        |   10.10.1.0/24  |    ALLOW   | ALLOW CONNECTION TO THE WEB SERVERS     |
|    _3991_    |    5601   |      TCP     | AzureLoadBalancer |  VirtualNetwork |    ALLOW   | ALLOW HEALTH PROBES FOR WEB SERVERS     |
|    _4001_    |     22    |      TCP     |      10.0.0.4     |  VirtualNetwork |    ALLOW   | ALLOW SSH CONNECTION TO OTHER MACHINES  |

**EXTRA** - The Elk requests are set to come through the HTTP port and forwarded to port 5601 by load balancer

**EXTRA** - To harden the system further some of the rules have been modified and added, here is the results:

| **PRIORITY** |  **PORT** | **PROTOCOL** |     **SOURCE**    | **DESTINATION** | **ACTION** |             **DESCRIPTION**             |
|:------------:|:---------:|:------------:|:-----------------:|:---------------:|:----------:|:---------------------------------------:|
|    _3990_    |     80    |      TCP     |  108.168.111.241  |   10.10.1.0/24  |    ALLOW   | ALLOW CONNECTION TO THE WEB SERVERS     |
|    _3991_    |    5601   |      TCP     | AzureLoadBalancer |  VirtualNetwork |    ALLOW   | ALLOW HEALTH PROBES FOR WEB SERVERS     |
|    _3992_    | 5601,9200 |      TCP     |   VirtualNetwork  |  VirtualNetwork |    ALLOW   | ALLOW NETWORK TO RECEIVE THE BEATS DATA |
|    _4001_    |     22    |      TCP     |      10.0.0.4     |  VirtualNetwork |    ALLOW   | ALLOW SSH CONNECTION TO OTHER MACHINES  |
|    _4096_    |    ANY    |      ANY     |        ANY        |       ANY       |    DENY    | DENY ALL TRAFFIC ON VNET                |


### SSH Key Configuration

For this project, the key-based ssh authentication is used and first step is to generate ssh keypair:

![Ssh Keygen](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/SSH-KEYGEN.png)


The rest of the machines are being configured from the ansible container that is installed on the jbox. The public key used to ssh on other virtual machines is generated on the jbox inside the ansible container.

The virtual machines can be setup all at once, but the ssh key would need to be reset for the ansible to be able to use the playbooks. Here is the way to do just that:

![Vm Reset](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VM-RESET.png)


### VM configuration

Next, we have configured virtual machines, all of the machines are created in this step and the same ssh public key is used in the process. The are three options for the VM sizes that we are using on the network.

![Vm Size](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VM-SIZES.png)

The sizes can be any of the available in the region, however the following are the minimum requirements for each of the machines:
- JUMP BOX: Standard_B1s
- DVWA SERVERS: Standard_B1ms
- ELK SERVERS: Standard_B2s

When creating the JBOX machine only settings that are selected, the resource group and inputting ssh key. For DVWA machines we are creating the availability zone, while for Elk machines we are using different region.

**EXTRA** - Create another availability zone for the two elk machines to ensure that the load balancer can be used with this set-up.

![Vm Availability](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VM-AVAILABILITY.png)

![Vm Ssh Key](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VM-SSH-KEY.png)

Lastly, in the networking option, the virtual network and subnetworks have been selected for the machines and public ip addresses are created for JBOX and ELK machines.

**EXTRA** - When using two elk server machines only JBOX public ip address will be created in this step

![Vm Network](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VM-NETWORK.png)


### Load Balancer Configuration

Once all of the machines are created th eload balancer is created to distribute the load on the DVWA servers. The basic tier load balancer is being used for this setup

**EXTRA** - Add second load balancer for the elk servers as well.

![Vm Network](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-TIER.png)

Create the public IP address for the DVWA and ELK to be accees through brower

![Vm Network](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-PUB-IP.png)

The virtual machines are added to the backend pool for the load balancer to forward the requests to

![Vm Network](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-BACK-END.png)

The rule can be created at the time of load balancer creation or after the resource was created. For the DVWA servers the rule will be listening on port 80 and forwarding the requests to port 80 as well.

![Vm Network](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-RULE.png)

Lastly, to ensure that the load balancer is functionig properly, the health probes are added to listen on port 80 to determine if the server available.

![Vm Network](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-HEALTH.png)

**EXTRA** - in the event that the load balancer is created for the elk servers, the health probes are set to listen on port 5601.

**EXTRA** - additionally, the ELK loadbalancer can be set to receive the requests on port 80 and forward them to port 5601.



### Public IP Configurations



### ANSIBLE Configuration

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

Last step to ensure that everything is ready for next configurations is to edit [Ansible](https://github.com/rrazumov-rrs/cyber-project/tree/main/ANSIBLE) configuration files:
- _[ansible config file](https://github.com/rrazumov-rrs/cyber-project/blob/main/ANSIBLE/ansible.cfg)_
- _[ansible hosts file](https://github.com/rrazumov-rrs/cyber-project/blob/main/ANSIBLE/hosts)_


### Using the Playbook

The following example of the Ansible playbook was created to show the syntax of the yml playbook:

Here is the example of [Basic](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS/upgrade-basic.yml) and [Advanced](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS/upgrade-advanced.yml) upgrade playbooks. 

The main note to take that all of the playbook files atart with three dashes(-) on the top line as well as all indentations are with two spaces.

### DVWA Configuration

_[dvwa setup playbook](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS)_


### ELK Configuration

_[elk setup playbook](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS)_


### BEATS Configuration

[Beats](https://github.com/rrazumov-rrs/cyber-project/blob/main/BEATS) configuration files:
- _[filebeat config file](https://github.com/rrazumov-rrs/cyber-project/blob/main/BEATS/filebeat-config.yml)_
- _[metsicbeat config file](https://github.com/rrazumov-rrs/cyber-project/blob/main/BEATS/metricbeat-config.yml)_

_[filebeat setup playbook](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS)_
_[metricbeat setup playbook](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS)_





















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


This ELK server is configured to monitor the following machines:
- _TODO: List the IP addresses of the machines you are monitoring_

We have installed the following Beats on these machines:
- _TODO: Specify which Beats you successfully installed_

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._


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