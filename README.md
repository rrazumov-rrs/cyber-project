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

Once all of the machines are created th eload balancer is created to distribute the load on the DVWA servers. The basic tier load balancer is being used for this setup.

**EXTRA** - Add second load balancer for the elk servers as well.

![Lb Tier](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-TIER.png)

Create the public IP address for the DVWA and ELK to be accees through brower. The IP addresses that are used in the project are set to static to prevent the machines from assigning new IP address everytime the machine reboots

![Lb Frontend](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-PUB-IP.png)

The virtual machines are added to the backend pool for the load balancer to forward the requests to

![Lb Backend](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-BACK-END.png)

The rule can be created at the time of load balancer creation or after the resource was created. For the DVWA servers the rule will be listening on port 80 and forwarding the requests to port 80 as well.

![Lb Rules](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-RULE.png)

Lastly, to ensure that the load balancer is functionig properly, the health probes are added to listen on port 80 to determine if the server available.

![Lb Health](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/LB-HEALTH.png)

**EXTRA** - in the event that the load balancer is created for the elk servers, the health probes are set to listen on port 5601.

**EXTRA** - additionally, the ELK loadbalancer can be set to receive the requests on port 80 and forward them to port 5601.


### **EXTRA** IP Configurations

The following configuration is used to create Domain Name Record for the DVWA and ELK load balancer to use in the browser:

![DNS Record](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/PUB-IP-DNS.png)

### ANSIBLE Configuration

Now that the virtual network is set up and running, the services on the machines are going to be configured. First the JBOX is configured to have ansible docker container runnig to automate the setup of other machines.

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

Next the ssh key is generated in the ansible container and all of the virtual machines have the ssh key reset in the settings

**EXTRA** - on the JBOX the ssh public key can be appended to the ~/.ssh/authorized_keys file in order to use ansible playbook for all machines on the network.

![Vm Reset](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/VM-RESET.png)

Last step to ensure that everything is ready for next configurations is to edit [Ansible](https://github.com/rrazumov-rrs/cyber-project/tree/main/ANSIBLE) configuration files:
- _[ansible config file](https://github.com/rrazumov-rrs/cyber-project/blob/main/ANSIBLE/ansible.cfg)_
- _[ansible hosts file](https://github.com/rrazumov-rrs/cyber-project/blob/main/ANSIBLE/hosts)_


### Using the Playbook

One the ansible setup and preparation is complete, the machines need to be configured to use DVWA docker container that is listening for the requests on port 80 as well as Elk server that is listening on ports 5044, 5601, and 9200.

This procedure can be done manually on each server, however, ansible container offers the automation in the form of the playbooks. Additionally, the use of docker containers along with ansible playbooks ensures that every container is set with the same parameters and provide identical performance across the installations.

The following example of the Ansible playbook was created to show the syntax of the yml playbook:

Here is the example of [Basic](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS/upgrade-basic.yml) and [Advanced](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS/upgrade-advanced.yml) upgrade playbooks.

The main note to take that all of the playbook files atart with three dashes(-) on the top line as well as all indentations are with two spaces.

![Ansible Tree](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/ANSIBLE-FOLDER-TREE.png)

Taking into consideration the structure of the /etc/ansible folder, the command to run upgrade playbook will be:
- ansible-palybook /etc/ansible/all-vm/upgrade.yml

After running the above command, ansible executes the playbook and runs all of the instructions in the playbook agains the 'host' list that is saved in /etc/ansible/hosts file.

To continue the setup for DVWA and ELK server the [dvwa-setup.yml](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS/dvwa-setup.yml) and [install-elk.yml](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS/install-elk.yml) playbooks are created and run inside the ansible container. This process might take a little bit of the time and at the end will give similar results.

![Playbook Execution](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/PLAY-RUN.png)

As long as the processes completed without any errors, the services are installed and runnig on the correct machines. In order to test that everything is functionning properly, web browser can be used to check both services:
- DVWA - 20.213.234.237/login.php
- ELK - 52.243.67.47/app/kibana

**EXTRA** - If the DNS records were created and Elk load balancer was set to listen on port 80, the result of the test will be similar to 

![DVWA web page](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/DVWA-RUNNING.png)

![Kibana web page](https://github.com/rrazumov-rrs/cyber-project/blob/main/IMAGES/KIBANA-RUNNING.png)

### BEATS Configuration

Once both DVWA and ELK services are running properly, it is the time to configure the monitoring of the DVW instances. The monitoring applications that we will be using are BEATS, to be specific FILEBEATS and METRICBEATS services in order to do that successfully, the configuratin yml files have to be configured; the details can be found [HERE](https://github.com/rrazumov-rrs/cyber-project/blob/main/BEATS) along with both [filebeat](https://github.com/rrazumov-rrs/cyber-project/blob/main/BEATS/filebeat-config.yml) and [metsicbeat](https://github.com/rrazumov-rrs/cyber-project/blob/main/BEATS/metricbeat-config.yml) configuration files.

NOTE: The configuration files are created on ansible machine and will be dropped on the target system as part of the playbook.

Lastly, the _[filebeat setup](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS/filebeat-on-dvwa.yml)_ and _[metricbeat setup](https://github.com/rrazumov-rrs/cyber-project/tree/main/PLAYBOOKS/metric-on-dvwa.yml)_ playbooks are created and have been successfully executed and it is time to check that the beats are working properly.

Kibana.webm
