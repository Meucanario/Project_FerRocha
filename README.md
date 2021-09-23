This document contains the following details:

Description of the Topology
Access Policies
ELK Configuration
Beats in Use
Machines Being Monitored
How to Use the Ansible Build
Description of the Topology
This repository includes code defining the infrastructure below.

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the "D*mn Vulnerable Web Application."

Load balancing ensures that the application will be highly available , in addition to restricting inbound access to the network. The load balancer ensures that the work will process incoming traffic will be shared by both vulnerable web servers. Access controls will ensure that only authorized users will be able to connect.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems of the Vms on the network ans well as watch system metrics , such as CPU usage,attempted SSH logins; 'sudo' escalation failures; etc.

The Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
The Metricbeat periodically collects metrics from the operating system and from services running on the server.
The configuration details of each machine may be found below.

Name	Function	IP Address	Operating System
Jump Box	Gateway		10.0.0.4	Linux
DVWA/VM 1	Web Server	10.0.0.5	Linux
DVWA/VM 2	Web Server	10.0.0.6	Linux
ELK VM		Monitoring	10.2.0.4	Linux
In additon Azure has provisioned a ** Load Balancer** in front of all machines except for the Jump Box.

Access Policies
The machines on the internal network are not exposed to the public Internet.

Only the jump box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:'52.173.32.30'

Machines within the network can only be accessed by each other. The DWVA 1 and DWVA 2 VMs send trafffic to the ELK server.

A summary of the access policies in place can be found in the table below.

Name		Publicly Accessible	Allowed IP Addresses
Jump Box		 Yes		52.173.32.30
ELK			 No		10.2.0.4
DVWA 1	                 No		10.0.0.5
DVWA 2			 No		10.0.0.6
Elk Configuration
Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it's easier to diploy and it allows for less human errors.

The playbook implements the following tasks:

TODO: In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc.
Install docker
Install DVWA image
Increase system memeory for DVWA and then enable docker module
The following screenshot displays the result of running docker ps after successfully configuring the ELK instance.

TODO: Update the path with the name of your screenshot of docker ps output

The playbook is duplicatied below.

---
# install_elk.yml
- name: Configure Elk VM with Docker
  hosts: elkservers
  remote_user: elk
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
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
Target Machines & Beats
This ELK server is configured to monitor the DWVA 1 and DVWA 2 VMs,at '10.0.0.5' and '10.0.0.6', respectively.

We have installed the following Beats on these machines:

-Filebeat -Metricbeat

These Beats allow us to collect the following information from each machine:

Filebeat: Filebeat detects changes to the filesystem. In Particular, we use it to collect Apache logs.
Metricbeat: Metricbeat detects changes in system metrics, such as CPU usage. We use it to detect SSH login attempts, failed sudo escalations, and CPU/RAM statistics.
The playbook below installs Metricbeat on the target hosts. The playbook for installing Filebeat is not included, but looks essentially identical — simply replace metricbeat with filebeat, and it will work as expected.

---
- name: Install metric beat
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
      src: /etc/ansible/files/metricbeat-config.yml
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
Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. We use the ** Jump Box** for this purpose.

Assuming you have such a control node provisioned:

SSH into the control node and follow the steps below:

Copy the playbook file to the Ansible Control Node.
Update the playbook file to include HOST.
Run the playbook, and navigate to etc/filebeat directory to check that the installation worked as expected.​TODO: Answer the following questions to fill in the blanks:
_Which file is the playbook? Where do you copy it?
Playbook is called - my-playbook.yml
located inside etc/ansible
Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on? HOSTS file.
_Which URL do you navigate to in order to check that the ELK server is running? http://10.2.0.4:5601 -Located inside the ansible playbook and set the hosts equal to the group I want to install it to.
_As a Bonus, provide the specific commands the user will need to run to download the playbook, update the files, etc.

wget github.com/playbook.yml - this will download the playbook into the ansible container.
Then nano playbook.yml to make edits.

The second method is using Github:
git pull
To update the files:
git add .
git commit -m 'message'
git push