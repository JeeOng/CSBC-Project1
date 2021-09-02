# Cybersecurity Bootcamp - PROJECT #1 - ELK Stack Server

## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![](https://github.com/JeeOng/Cybersec_Bootcamp/blob/main/Diagrams/Network%20Diagram.JPG)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _____ file may be used to install only certain pieces of it, such as Filebeat.

#### Playbook 1: pentest.yml
```
---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present
  - name: Install pip3
      apt:
        name: python3-pip
        state: present
  - name: Install Docker python module
      pip:
        name: docker
        state: present
  - name: download and launch a docker web container
      docker_container:
        name: dvwa
        image: cyberxsecurity/dvwa
        state: started
        restart_policy: always
        published_ports: 80:80
  - name: Enable docker service
      systemd:
        name: docker
        enabled: yes

```

#### Playbook 2: install-elk.yml
```
---
- name: Configure Elk VM with Docker
  hosts: elk
  become: true
  tasks:
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
    - name: Install Docker module
      pip:
        name: docker
        state: present
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: 262144
        state: present
        reload: yes
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```

#### Playbook 3: filebeat-playbook.yml
```
---
- name: installing and launching filebeat
  hosts: elk
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

  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
```

#### Playbook 4: metricbeat-playbook.yml
```
---
- name: installing and launching metricbeat
  hosts: elk
  become: yes
  tasks:
  - name: download metricbeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

  - name: install metricbeat deb
    command: dpkg -i metricbeat-7.4.0-amd64.deb

  - name: drop in metricbeat.yml
    copy:
      src: /etc/ansible/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure docker module
    command: metricbeat modules enable docker

  - name: setup metricbeat
    command: metricbeat setup

  - name: enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
```

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build

### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting all access to the network.
- Load balancing helps protect application availability similarly to a firewall to allow requests from clients to be shared across multiple servers.
- The Jump box via the Ansible container provides an advantage to minimise potential attack surfaces, and is a secure computer admins can connect to before launching administrative tasks to negate an origin point in remote connections.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the configurating and system files & logs.
- Filebeat is used to help monitor log files
- Metricbeat is used to help collect service statistics from monitored Virtual Networks (VMs).

The configuration details of each machine may be found below.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.4   | Linux            |
| Web-1    | DVWA     | 10.0.0.8   | Linux            |
| Web-2    | DVWA     | 10.0.0.9   | Linux            |
| Web-3    | DVWA     | 10.0.0.11  | Linux            |
| ELK-VM   | ELK      | 10.2.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump-Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 191.239.180.26

Machines within the network can only be accessed by Jump Box.
- Access to the ELK VM can be done through the Jump Box Provisioner VM. The Jump Box's IP is: 10.0.0.4

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 191.239.180.26       |
| Web-1    | Yes                 | 23.101.228.243       |
| Web-2    | Yes                 | 23.101.228.243       |
| Web-3    | Yes                 | 23.101.228.243       |
| ELK-VM   | Yes                 | 20.37.240.48         |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- The main advantage of automating configuration with Ansible is to assist IT admins with an all-in-one packet rollout quickly and consistently

The playbook implements the following tasks:
- Install Docker.io
- Install Python3-pip
- Install Docker Python Module
- Increase Virtual Memory
- Download and Launch ELK Container
- Enable Docker Service

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![](https://github.com/JeeOng/Cybersec_Bootcamp/blob/main/Diagrams/ELK-Stack_DockerPS.jpg)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- **Web-1** 10.0.0.8
- **Web-2** 10.0.0.9
- **Web-3** 10.0.0.11

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeat: Allows us to collect system log information from each machine.
- Metricbeat: Collects statistical information from the operating system and services.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook file to docker container.
- Update the Ansible file to include [IP Address] ansible_python_interpreter=/usr/bin/python3
- Run the playbook, and navigate to the VM to check that the installation worked as expected.
