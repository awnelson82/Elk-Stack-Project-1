# Elk-Stack-Project-1

## ELK-Stack-Project
**Automated ELK Stack Deployment**
 
The files in this repository were used to configure the network depicted below.

![vNet Diagram](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Diagrams/ELK-Project-V1.jpg)
 
These files have been tested and used to generate an automated ELK Stack Deployment on Azure. They can be used to either recreate the entire deployment figured below. Otherwise, select portions of the YAML files may be used to install only certain pieces of it, for example, Filebeat and Metricbeat.

  - [install-elk.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/install-elk.yml)
  - [filebeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/filebeat-config.yml)
  - [filebeat-playbook.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/filebeat-playbook.yml)
  - [metricbeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/metricbeat-config.yml)
  - [metricbeat-playbook.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/metricbeat-playbook.yml)
 
This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
- Beats in Use
- Machines Being Monitored
- How to Use the Ansible Build
 
### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network.

> What aspect of security do load balancers protect?
- According to [Azure security baseline for Azure Load Balancer](https://bit.ly/3AnSRPV), the load balancer's main purpose is to distribute web traffic across multiple servers. In our network, the load balancer was installed in front of the VM to 
   - protect Azure resources within virtual networks.
   - monitor and log the configuration and traffic of virtual networks, subnets, and NICs.
   - protect critical web applications
   - deny communications with known malicious IP addresses
   - record network packets
   - deploy network-based intrusion detection/intrusion prevention systems (IDS/IPS)
   - manage traffic to web applications
   - minimize complexity and administrative overhead of network security rules
   - maintain standard security configurations for network devices
   - document traffic configuration rules
   - use automated tools to monitor network resource configurations and detect changes


> What is the advantage of a jump box?
- A Jump Box or a "Jump Server" is a gateway on a network used to access and manage devices in different security zones. A Jump Box acts as a "bridge" between two trusted networks zones and provides a controlled way to access them. We can block the public IP address associated with the VM. It helps to improve security also prevents all Azure VM’s to expose to the public.

Integrating an Elastic Stack server allows us to easily monitor the vulnerable VMs for changes to their file systems and system metrics such as privilege escalation failures, SSH logins activity, CPU and memory usage, etc.

> What does Filebeat watch for?
- Filebeat helps keep things simple by offering a lightweight way (low memory footprint) to forward and centralize logs, files and watches for changes.

> What does Metricbeat record?
- Metricbeat helps monitor servers by collecting metrics from the system and services running on the server so it records machine metrics and stats, such as uptime.

The configuration details of each machine may be found below.
 
| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump-Box-Provisioner | Gateway  | 44.77.55.33 ; 10.0.0.4   | Linux            |
| Web-1        |webserver    | 10.0.0.5     | Linux            |
| Web-2        |webserver    | 10.0.0.6     | Linux            |
| ELKServer    |Kibana       | 104.45.159.216 ; 10.1.0.4     | Linux            |
| RedTeam-LB|Load Balancer| 40.122.215.16| DVWA            |
 
In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into availability zones: Web-1 + Web-2


### Access Policies
 
The machines on the internal network are not exposed to the public Internet.
 
Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses: 47.185.204.83 Machines within the network can only be accessed by SSH from Jump Box.
 
A summary of the access policies in place can be found in the table below.
 
| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump-Box-Provisioner | Yes                 | 47.185.204.83        |
| ELKServer      | Yes                  |  47.185.204.83:5601        |
| DVWA 1   | No                  |  10.0.0.1-254        |
| DVWA 2   | No                  |  10.0.0.1-254        |


 
---


### ELK Configuration
 
Ansible was used to automate the configuration of the ELK server. No configuration was performed manually, which is advantageous because Ansible can be used to easily configure new machines, update programs, and configurations on hundreds of servers at once, and the best part is that the process is the same whether we're managing one machine or dozens and even hundreds.

> What is the main advantage of automating configuration with Ansible?
- Ansible is focusing on bringing a server to a certain state of operation.

<details>
<summary> <b> Click here to view ELK Configuration. </b> </summary>

---
 
We will configure an ELK server within virtual network. Specifically,
 
- Deployed a new VM on our virtual network.
- Created an Ansible play to install and configure an ELK instance.
- Restricted access to the new server.

#### Deployed a new VM on our virtual network. 
 
1. Create a new vNet located in the same resource group we have been using. 
- Make sure this vNet is located in a new region and not the same region as our other VM's, which region we select is not important as long as it's a different US region than our other resources, we can also leave the rest of the settings at default.
- In this example, that the IP Addressing has automatically created a new network space of 10.1.0.0/16. If our network is different (10.2.0.0 or 10.3.0.0) it is ok as long as we accept the default settings. Azure automatically creates a network that will work.

![Create vNet](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Create%20vNet.png)  

2. Create a Peer connection between our vNets. This will allow traffic to pass between our vNets and regions. This peer connection will make both a connection from our first vNet to our second vNet and a reverse connection from our second vNet back to our first vNet. This will allow traffic to pass in both directions.
- Navigate to `Virtual Network` in the Azure Portal.
- Select our new vNet to view it's details.
- Under `Settings` on the left side, select `Peerings`.
- Click the + Add button to create a new Peering.
- A unique name of the connection from our new vNet to our old vNet such as depicted example below.
- Choose our original RedTeam vNet in the dropdown labeled `Virtual Network`.
- Leave all other settings at their defaults.
 
![PeeringsELKtoRed](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/ELKtoRed.png) 
 
![PeeringsRedtoELK](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/RedtoELK.png)  

3. Create a new Ubuntu VM in our virtual network with the following configurations:
- The VM must have a public IP address.
- The VM must be added to the new region in which we created our new vNet. We want to make sure we select our new vNEt and allow a new basic Security Group to be created for this VM.
- The VM must use the same SSH keys as our WebserverVM's. This should be the ssh keys that were created on the Ansible container that's running on our jump box.
- After creating the new VM in Azure, verify that it works as expected by connecting via SSH from the Ansible container on our jump box VM.

   - ```bash
        ssh sysadmin@<jump-box-provisioner>
     ``` 
   - ```bash
        sudo docker container list -a
     ``` 
   - ```bash
        sudo docker start goofy_wright && sudo docker attach goofy_wright
     ``` 
 
![connect_on_newVM](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/connect_on_newVM.png)  
 
- Copy the SSH key from the Ansible container on our jump box:
   - RUN `cat id_rsa.pub` Configure a new VM using that SSH key.
 
![RSA](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/id_rsa.pub_on_newVM.png) 
 

#### Created an Ansible play to install and configure an ELK instance.

In this step, we have to:
- Add our new VM to the Ansible hosts file.
- Create a new Ansible playbook to use for our new ELK virtual machine.
- From our Ansible container, add the new VM to Ansible's hosts file.
   - RUN `nano /etc/ansible/hosts` and put our IP with `ansible_python_interpreter=/usr/bin/python3`

![hosts file editing](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/CatHosts.png)  

- In the below play, representing the header of the YAML file, I defined the title of my playbook based on the playbook's main goal by setting the keyword 'name:' to: "Configure Elk VM with Docker". Next, I defined the user account for the SSH connection, by setting the keyword 'remote_user:' to "sysadmin" then activated privilege escalation by setting the keyword 'become:' to "true". 
 
 The playbook implements the following tasks:

```yaml
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: sysadmin
  become: true
  tasks:
```
 
In this play, the ansible package manager module is tasked with installing docker.io. The keyword 'update_cache:' is set to "yes" to download package information from all configured sources and their dependencies prior to installing docker, it is necessary to successfully install docker in this case. Next the keyword 'state:' is set to "present" to verify that the package is installed.


```yaml
     # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present
```

In this play, the ansible package manager module is tasked with installing  'pip3', a version of the 'pip installer' which is a standard package manager used to install and maintain packages for Python.
The keyword 'force_apt_get:' is set to "yes" to force usage of apt-get instead of aptitude. The keyword 'state:' is set to "present" to verify that the package is installed.

```yaml
      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
```

In this play the pip installer is used to install docker and also verify afterwards that docker is installed ('state: present').

```yaml
      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
```

In this play, the ansible sysctl module configures the target virtual machine (i.e., the Elk server VM) to use more memory. On newer version of Elasticsearch, the max virtual memory areas is likely to be too low by default (ie., 65530) and will result in the following error: "elasticsearch | max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]", thus requiring the increase of vm.max_map_count to at least 262144 using the sysctl module (keyword 'value:' set to "262144"). The keyword 'state:' is set to "present" to verify that the change was applied. The sysctl command is used to modify Linux kernel variables at runtime, to apply the changes to the virtual memory variables, the new variables need to be reloaded so the keyword 'reload:' is set to "yes" (this is also necessary in case the VM has been restarted).

```yaml
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
```

In this play, the ansible docker_container module is used to download and launch our Elk container. The container is pulled from the docker hub repository. The keyword 'image:' is set with the value "sebp/elk:761", "sebp" is the creator of the container (i.e., Sebastien Pujadas). "elk" is the container and "761" is the version of the container. The keyword 'state:' is set to "started" to start the container upon creation. The keyword 'restart_policy:' is set to "always" and will ensure that the container restarts if we restart our web vm. Without it, we will have to restart our container when we restart the machine.
The keyword 'published_ports:' is set with the 3 ports that are used by our Elastic stack configuration, i.e., "5601" is the port used by Kibana, "9200" is the port used by Elasticsearch for requests by default and "5400" is the default port Logstash listens on for incoming Beats connections (we will go over the Beats we installed in the following section "Target Machines & Beats").

```yaml
      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```

In this play, the ansible systemd module is used to start docker on boot, setting the keyword 'enabled:' to "yes".

```yaml
      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```
![Install_elk_yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Install_elk_yml.png)

Now we can start launching and exposing the container by run

```bash
ansible-playbook install-elk.yml
```

The following screenshot displays the result of running `install-elk.yml`

![Docker ELKResult output](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Install_elk_result.png)

SSH to our container: ```ssh sysadmin@10.1.0.4``` and RUN ```sudo docker ps```

The following screenshot displays the result of running `docker ps` after successfully configuring the Elastic Stack instance.

![Docker InstallELK output](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/InstallELK.png)

Logging into the Elk server and manually launch the ELK container with: 

```bash
sudo docker start elk
```
then ```curl http://localhost:5601/app/kibana``` does return HTML.

The following screenshot displays the result of running `curl` after start ELK container

![Docker curl output](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/CurlResult.png)

#### Restricted access to the new server.
	
This step is to restrict access to the ELK VM using Azure's network security groups (NSGs). We need to add public IP address to a whitelist, just as we did when clearing access to jump box.

Go to Network Security Group to config our host IP to Kibana as follow

![Docker InboundSecRules output](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Docker%20InboundSecRules%20output.png)

Then try to access web browser to http://<your.ELK-VM.External.IP>:5601/app/kibana 
 
![Access_Kibana](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Access_Kibana.png)

</details>

---

### Target Machines & Beats
This ELK server is configured to monitor the following machines:

- Web-1 (DVWA 1) | 10.0.0.5
- Web-2 (DVWA 2) | 10.0.0.6

I have installed the following Beats on these machines:

- Filebeat
- Metricbeat

<details>
<summary> <b> Click here to view Target Machines & Beats. </b> </summary>

---

	
These Beats allow us to collect the following information from each machine:

`Filebeat`: Filebeat detects changes to the filesystem. I use it to collect system logs and more specifically, I use it to detect SSH login attempts and failed sudo escalations.

We will create a [filebeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/filebeat-config.yml) and [metricbeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/metricbeat-config.yml) configuration files, after which we will create the Ansible playbook files for both of them.

Once we have this file on our Ansible container, edit it as specified:
- The username is elastic and the password is changeme.
- Scroll to line #1106 and replace the IP address with the IP address of our ELK machine.
output.elasticsearch:
hosts: ["10.1.0.4:9200"]
username: "elastic"
password: "changeme"
- Scroll to line #1806 and replace the IP address with the IP address of our ELK machine.
	setup.kibana:
host: "10.1.0.4:5601"
- Save both files filebeat-config.yml and metricbeat-config.yml into `/etc/ansible/files/`

![files_FMconfig](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/files_FMconfig.png) 
 
 
Next, create a new playbook that installs Filebeat & Metricbeat, and then create a playbook file, `filebeat-playbook.yml` & `metricbeat-playbook.yml`

RUN `nano filebeat-playbook.yml` to enable the filebeat service on boot by Filebeat playbook template below:

```yaml
---
- name: Install and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb
    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb
    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/roles/install-filebeat/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml
    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system
    # Use command module
  - name: Setup filebeat
    command: filebeat setup
    # Use command module
  - name: Start filebeat service
    command: service filebeat start
    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes

```

![Filebeat_playbook](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Filebeat_playbook.png) 
 
- RUN `ansible-playbook filebeat-playbook.yml`

![Filebeat_playbook_result](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Filebeat_playbook_result.png)  

Verify that our playbook is completed by navigate back to the Filebeat installation page on the ELK server GUI

![Filebeat_playbook_verify](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Filebeat_playbook_verify.png)
	
![Filebeat_playbook_verify1](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Filebeat_playbook_verify1.png)
		
	
`Metricbeat`: Metricbeat detects changes in system metrics, such as CPU usage and memory usage.

RUN `nano metricbeat-playbook.yml` to enable the metricbeat service on boot by Metricbeat playbook template below:

```yaml
---
- name: Install and Launch Metricbeat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb
    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb
    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/roles/install-metricbeat/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml
    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker
    # Use command module
  - name: setup metric beat
    command: metricbeat setup
    # Use command module
  - name: start metric beat
    command: service metricbeat start
    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
```

![Metricbeat_playbook](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Metricbeat_playbook.png)  
 
- RUN `ansible-playbook metricbeat-playbook.yml`

![Metricbeat_playbook_result](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Metricbeat_playbook_result.png)  

Verify that this playbook is completed by navigate back to the Filebeat installation page on the ELK server GUI

![Metricbeat_playbook_verify](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Metricbeat_playbook_verify.png)

 
</details>

---
 
### Using the Playbook

Next, I want to verify that `filebeat` and `metricbeat` are actually collecting the data they are supposed to and that my deployment is fully functioning.

To do so, I have implemented 3 tasks:

1. Generate a high amount of failed SSH login attempts and verify that Kibana is picking up this activity.
2. Generate a high amount of CPU usage on my web servers and verify that Kibana picks up this data.
3. Generate a high amount of web requests to my web servers and make sure that Kibana is picking them up.
	
<details>
<summary> <b> Click here to view Using the Playbook. </b> </summary>

---

