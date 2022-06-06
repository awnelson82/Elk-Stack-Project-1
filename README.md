# Elk-Stack-Project-1
**Automated ELK Stack Deployment**
 
The files in this repository were used to configure the network depicted below.

![vNet Diagram](https://github.com/awnelson82/Elk-Stack-Project-1/blob/main/Diagrams/homework12%20(2).drawio.png)
 

  - [install-elk.yml](https://github.com/awnelson82/Elk-Stack-Project-1/blob/main/Ansible/install-elk.yml)
  - [filebeat-config.yml](https://github.com/awnelson82/Elk-Stack-Project-1/blob/main/Ansible/filebeat-configuration.yml)
  - [filebeat-playbook.yml](https://github.com/awnelson82/Elk-Stack-Project-1/blob/main/Ansible/filebeat-playbook.yml)
  - [metricbeat-config.yml](https://github.com/awnelson82/Elk-Stack-Project-1/blob/main/Ansible/metricbeat-configuration.yml)
  - [metricbeat-playbook.yml](https://github.com/awnelson82/Elk-Stack-Project-1/blob/main/Ansible/metricbeat-playbook.yml)
 
This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
- Beats in Use
- Machines Being Monitored
- How to Use the Ansible Build
 
### Description of the Topology

The purpose of this network is to show a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, as well as, restricting inbound access to the network.



> What aspect of security do load balancers protect?
- Load balancers protect networks and organizations against DDoS attacks by distributing web traffic across multiple servers. They allow servers to be moved from the organization's servers to a public cloud provider.
	
In our network, the load balancer was installed in front of the VM to 
   - protect Azure resources within virtual networks.
   - monitor and log the configuration and traffic of virtual networks, subnets, and NICs.
   - protect critical web applications
   - deny communications with known malicious IP addresses
   - record network packets
   - deploy network-based intrusion detection/intrusion prevention systems (IDS/IPS)
   - manage traffic to web applications
   - maintain standard security configurations for network devices
   - use automated tools to monitor network resource configurations and detect changes


> What is the advantage of a jump box?
- A jump box allows a machine to exist that controls access to other devices or servers on a network. It serves as a bridge between trusted networks to provide a controlled access point to the networks. It also allows the administrator to allow access or deny to certain devices and/or people, adding layers of security.

>Integrating an Elastic Stack server allows us to easily monitor the vulnerable VMs for changes to their file systems and system metrics such as privilege escalation failures, SSH logins activity, CPU and memory usage, etc.

> What does Filebeat watch for?
- Filebeat watches for log files and collects log events and forwards them to Elasticsearch or Logstash to be indexed.

> What does Metricbeat record?
- Meticbeat monitors and collects metrics and statistics about your website and ships them to an output such as Elasticsearch or Logstash.

The configuration details of each machine may be found below.
 







| \| Name     \| Function \| IP Address \| Operating System \| |
|--------------------------------------------------------------|
| \|----------\|----------\|------------\|------------------\| |
|                                                              
| \| Jump Box   \| Gateway      \| 10.0.0.4   \| Linux            \| |
|                                                              
| \| Web-1 VM  \| Web Server \| 10.0.0.5   \| Linux            \| |
|                                                              
| \| Web-2 VM  \| Web Server \| 10.0.0.6   \| Linux            \| |
|                                                              
| \| ElkVM         \| Elk Server   \| 10.1.0.6   \| Linux            \| |


 
In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into availability zones: Web-1 + Web-2


### Access Policies
 
The machines on the internal network are not exposed to the public Internet.
 
Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:My personal IP addr. Machines within the network can only be accessed by SSH from Jump Box.
 
A summary of the access policies in place can be found in the table below.
 
| \| Name     \| Publicly Accessible \| Allowed IP Addresses \| |
|---------------------------------------------------------------|
|
| \|----------\|---------------------\|----------------------\| |
|                                                               
| \| Jump Box  \| Yes                 \|  My personal IP addr  \| |
|                                                               
| \|Web-1 VM  \| No                  \|  10.0.0.5 by SSH      \| |
|                                                               
| \|Web-2 VM  \| No                  \|  10.0.0.6 by SSH      \| |
|                                                               
| \|Elk VM        \| No                  \| 10.1.0.6 by SSH


 
---


### ELK Configuration
 
Ansible was used to automate the configuration of the ELK server. No configuration was performed manually, which is advantageous because Ansible can be used to easily configure new machines, update programs, and configurations on hundreds of servers at once and the process is the same whether we're managing one machine or multiple machines

> What is the main advantage of automating configuration with Ansible?
- It is very simple to set up and to use. You do not need special coding skills to use Ansible playbooks. It is powerful. You can model highly complex IT workflows. It is flexible. You can orchestrate an entire application environment no matter where it is deployed.

The playbook implements the following tasks:

- Install docker.io
- Install Python-pip
- Install docker container
- Download and launch docker web container


The following screenshot displays the result of running `docker ps` after successfully configuring the Elastic Stack instance.



---

### Target Machines & Beats
This ELK server is configured to monitor the following machines:

- Web-1 (DVWA 1) | 10.0.0.5
- Web-2 (DVWA 2) | 10.0.0.6

I have installed the following Beats on these machines:

- Filebeat
- Metricbeat


---

	
These Beats allow us to collect the following information from each machine:

- Filebeat: Filebeat monitors log files, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.

- Metricbeat: Metribeat collects metrics from the operating system and from services running on the server.

 
### Using the Playbook

In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:

- Copy the filebeat playbook file to /etc/ansible.
- Update the the configuration file to include the webservers and ElkVM
- Run the playbook, and navigate to ElkVM to check that the installation worked as expected. /etc/ansible/host should include:

---

_TODO: Answer the following questions to fill in the blanks:
- _Which file is the playbook? Where do you copy it?_
     - /etc/ansible/file/filebeat-configuration.yml

- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?
     - Edit the /etc/ansible/hosts file to add webserver/elkserver ip addresses

- _Which URL do you navigate to in order to check that the ELK server is running?
     - http://[my.ELK-VM.External.IP]:5601/app/kibana
