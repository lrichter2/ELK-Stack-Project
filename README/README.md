## Automated ELK Stack Deployment

https://docs.google.com/document/d/1fCsbBMxdFGaPR9lhlrusqw07X-0ONF1yMmZnbcgzoL0/edit?usp=sharing

**Automated ELK Stack Deployment**

The files in this repository were used to configure the network depicted below.

https://github.com/lrichter2/ELK-Stack-Project/blob/main/README/Images/ELK-Stack-Project%20Network%20Diagram%20(1).pdf

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the install-elk.yml file may be used to install only certain pieces of it, such as Filebeat.

https://github.com/lrichter2/ELK-Stack-Project/blob/main/Ansible/install-elk.yml

This document contains the following details:

1.  Description of the topology  
2.  Access Policies  
3.  ELK Configuration  
4.  Beats in Use  
5.  Machines Being Monitored  
6.  How to Use the Ansible Build  

**Description of the Topology**

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network.  The load balancer ensures that work to process incoming traffic will be shared by both vulnerable web servers.  Access controls will ensure that only authorized users, ourselves in this case, will be able to connect in the first place.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems of the VMs on the network, as well as watch system metrics, such as CPU usage, attempted SSH logins, sudo escalation failures, etc.

+What does Filebeat watch for?  Filebeat logs information about the file system such as which files have changed and when they changed.
+What does Metricbeat record?  Metricbeat collects metrics from the OS and from services running on the server.  It takes the metrics and statistics and ships the info to Elasticsearch or Logstash, for example.

The configuration details of each machine may be found below.

| Name       | Function   | IP Address | Operating System |
|------------|------------|------------|------------------|
| Jump Box   | Gateway    | 10.0.0.4   | Linux            |
| DVWA 1     | Web Server | 10.0.0.5   | Linux            |
| DVWA 2     | Web Server | 10.0.0.6   | Linux            |
| ELK-SERVER | Monitoring | 10.1.0.4   | Linux            |

In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box.  The load balancer’s targets are organized into the following availability zones.

..Availability Zone 1:  DVWA 1 + DVWA 2
..Availability Zone 2:  ELK

**ELK Server Configuration**

The ELK VM exposes an Elastic Stack instance. Docker is used to download and manage an ELK container.

Rather than configure ELK manually, we opted to develop a reusable Ansible Playbook to accomplish the task. This playbook is duplicated below.

To use this playbook, one must log into the Jump Box, then issue: ansible-playbook install_elk.yml elk. This runs the install_elk.yml playbook on the elk host.

**Access Policies**

The machines on the internal network are not exposed to the public Internet.

Only the jump box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP address:

..104.218.222.177

Machines within the network can only be accessed by each other.  The DVWA1 and DVWA 1 VMs send traffic to the ELK server.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 104.218.222.177      |
| ELK      | No                  | 10.1.0.1-254         |
| DVWA 1   | No                  | 10.0.0.1-254         |
| DVWA 2   | No                  | 10.0.0.1-254         |

**Elk Configuration**

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it drastically reduces the chance of human error.  Plus, multiple commands can be pushed to multiple machines at one time.

The playbook implements the following tasks:

  Install docker
  Install pip3
  Install docker python module
  Increase virtual memory
  Use more memory
  Download and launch a docker ELK container
  
The following screenshot displays the result of running docker ps after successfully configuring the ELK instance:

https://github.com/lrichter2/ELK-Stack-Project/blob/main/README/Images/sudo%20docker%20ps.jpg

The playbook is duplicated below:

https://github.com/lrichter2/ELK-Stack-Project/blob/main/Ansible/install-elk.yml

---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azadmin
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present
 
      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
 
      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
 
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
 
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
 
      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
 
**Target Machines & Beats**

This ELK server is configured to monitor the following machines:

  DVWA 1:  10.0.0.5
  DVWA 2:  10.0.0.6
  
We have installed the following Beats on these machines:

..Filebeat

These Beats allow us to collect the following information from each machine:

..Filebeat:  Filebeat detects changes to the file system.  Specifically, we use it to collect Apache logs.

The playbook below installs Filebeat on the target hosts.  The playbook for installing Metricbeat is not included, but you can simply replace filebeat with metricbeat and it will work as expected.

https://github.com/lrichter2/ELK-Stack-Project/blob/main/Ansible/filebeat-playbook.yml

---
- name: Installing and Launch Filebeat
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
      src: /etc/ansible/files/filebeat-config.yml
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
 

**Using the Playbook**

In order to use the playbook, you will need to have an Ansible control node already configured. We use the jump box for this purpose.  
To use the playbooks, you will perform the following steps:

..Copy the playbooks to the Ansible Control Node (container).
..Run each playbook on the appropriate targets.

The easiest way to copy the playbooks is to use Git.

$ cd /etc/ansible
$ mkdir files
# Clone Repository + IaC Files
$ git clone https://github.com/lrichter2/ELK-Stack-Project.git
# Move Playbooks and hosts file Into `/etc/ansible`
$ cp project-1/playbooks/* .
$ cp project-1/files/* ./files

This copies the playbook files to the correct place.
Next, a hosts file must be created to specify which VMs to run each playbook on. Run the commands below:

$ cd /etc/ansible
$ cat > hosts <<EOF
[webservers]
10.0.0.5
10.0.0.6

[elk]
10.1.0.4
EOF

After this, use the commands below to run the playbook:

$ cd /etc/ansible
$ ansible-playbook install_elk.yml elk
$ ansible-playbook install_filebeat.yml webservers

To verify success, wait five minutes to give ELK time to start up.

Then, attempt to access Kibana at http://52.176.105.94:5601.  If the installation succeeded, you should see the landing page.

