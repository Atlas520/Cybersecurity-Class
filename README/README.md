## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

https://drive.google.com/file/d/1FxGVeimCNLzs3VBYYhZYj6X0rHElmEMt/view?usp=sharing

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YAML file may be used to install only certain pieces of it, such as Filebeat.

---
- name: Configure Elk VM with Docker
  hosts: elk
#  remote_user: azadmin
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present
      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present
      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144
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
        # Please list the ports that ELK runs on
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build

---
- name: my first playbook
  hosts: webservers
  become: true
  tasks:


  - name: install docker
    apt:
      update_cache: yes
      name: docker.io
      state: present

  - name: install python3-pip
    apt:
      update_cache: yes
      name: python3-pip
      state: present

  - name: install docker pip
    pip:
      name: docker
      state: present

  - name: install dvwa container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      restart_policy: always
      published_ports: 80:80

  - name: enable docker service
    systemd:
      name: docker
      enabled: yes

---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start






### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
What aspect of security do load balancers protect? Availability of data accessed at all times meant to protect from flooding or being hit with DoS.
What is the advantage of a jump box? The jump box is used to deploy IAC to the rest of your network/other VMs. It is secured and used only in conjunction with your IP and an SSH Key so only you will be able to make changes to the other VMs on the network. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the Logs and system Traffic.
What does Filebeat watch for? Log files/ log events.
What does Metricbeat record? Metrics and Statistical data.

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function   | IP Address | Operating System |
|----------|------------|------------|------------------|
| Jump Box | Gateway    | 10.0.0.7   | Linux            |
| Web 1    | Web Server | 10.0.0.8   | Linux            |
| Web 2    | Web Server | 10.0.0.9   | Linux            |
| Web 3    | Web Server | 10.0.0.10  | Linux            |
| ELK      | Monitoring | 10.1.0.4   | Linux            |



### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
24.217.91.208
 

Machines within the network can only be accessed by Secure Shell (SSH).
Which machine did you allow to access your ELK VM? Jump Box. What was its IP address? 10.0.0.7

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses   |
|----------|---------------------|------------------------|
| Jump Box | Yes                 | 24.217.91.208          |
| Web 1    | No                  | 10.0.0.7               |
| Web 2    | No                  | 10.0.0.7               |
| Web 3    | No                  | 10.0.0.7               |
| ELK      | No                  | 10.0.0.7               |



### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
What is the main advantage of automating configuration with Ansible? The ease of use. You can create playbooks and copy them to each machine needed in seconds, you could scale up or down depending on business needs for creating VMs/webservers etc.

The playbook implements the following tasks:
-  Install docker.io
-  Install python3-pip
- Install Docker Module
- Increase/Use Virtual Memory
- Download and Launch Docker

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

https://docs.google.com/document/d/1Cdb9iFDAeBMYzDqDj1jF96i8Y_NUlIr2GjMleafgPp4/edit?usp=sharing

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
Web 1 10.0.0.8
Web 2 10.0.0.9
Web 3 10.0.0.10

We have installed the following Beats on these machines:
Filebeat on Web 1, Web 2, & Web 3

These Beats allow us to collect the following information from each machine:
Filebeat can collect data of users who connect to our web servers. An example of the data collected from this is country of origin and the OS used to connect. We were able to see that we had nearly 300 connections from China with 68 of them using MAC OSX.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook.yml file to /etc/ansible.
- Update the hosts file to include… which groups you want to deploy it to such as:
       [webservers]
10.0.0.9 ansible_python_interpreter=/usr/bin/python3
10.0.0.8 ansible_python_interpreter=/usr/bin/python3
10.0.0.10 ansible_python_interpreter=/usr/bin/python3
       [elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3


- Run the playbook, and navigate to [ELK Public ip]:5601/app/kibana to check that the installation worked as expected.

Answer the following questions to fill in the blanks:
- _Which file is the playbook? filebeat-playbook.yml Where do you copy it? /etc/ansible/roles
- _Which file do you update to make Ansible run the playbook on a specific machine? /etc/ansible/hosts. How do I specify which machine to install the ELK server on versus which to install Filebeat on? Specify the group [webservers] or [elk] in the playbook
- _Which URL do you navigate to in order to check that the ELK server is running? http://http://23.101.142.133:5601/app/kibana

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

