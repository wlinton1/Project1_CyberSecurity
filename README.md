## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![TODO: Update the path with the name of your diagram](Images/diagram_filename.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _playbook_ file may be used to install only certain pieces of it, such as Filebeat.

  - _TODO: Enter the playbook file._

  _elk-install.yml_   -- listed below -- at /etc/ansible/elk-install.yml

  ---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azadmin
  become: true
  tasks:
    
  - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

  - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

  - name: Install Docker python module
      pip:
        name: docker
        state: present

  - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes


        _filebeat-playbook.yml listed below_

        ---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: Enable and Configure System Module
    command: filebeat modules enable system

  - name: Setup filebeat
    command: filebeat setup

  - name: Start filebeat service
    command: service filebeat start

  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes

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

  - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes




This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly functional, in addition to restricting high traffic to the network.
- _TODO: What aspect of security do load balancers protect? What is the advantage of a jump box?_ 
--- load balancers protect the system from DDos attacks and keep traffic flowing evenly between servers such as web-1 web-2 web-3, it will restrict high traffic to the network. They prevent overloading and keep things running smoothly. 
-- jump box allows access to web-1 web-2 etc. and allows these to be secured and watched. you will use your jumpbox to connect to everything.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _____ and system _____.
- _TODO: What does Filebeat watch for?_
--- filebeat monitors log files or locations that are specified, collects log events and sends them to elsticsearch or logstash
- _TODO: What does Metricbeat record?_
--- metricbeat records metrics/statistics that it collects and sends them to your specified output such as elisticsearch or logstash. Metricbeat monitors servers by collecting metrics from the system or services running on your server

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.1   | Linux            |
| Web-1    | server   | 10.0.0.8   | Linux            |
| Web-2    | server   | 10.0.0.9   | Linux            |
| Web-3    | server   | 10.0.0.10  | Linux            |
| Elk-web  | server   | 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the _____ machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _TODO: Add whitelisted IP addresses_
--- 104.49.205.186 localhost ip

Machines within the network can only be accessed by _____.
- _TODO: Which machine did you allow to access your ELK VM? What was its IP address?_

Jump-Box-Provisioner 10.0.0.8 (private)

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 10.0.0.1 10.0.0.2    |
| ELK-Web  |  No                 |  10.0.0.8            |
| Web-1    |  no                 |  10.0.0.8            |
| Web-2    |  no                 |  10.0.0.8            |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _TODO: What is the main advantage of automating configuration with Ansible?_ YAML playbooks allow the quickest way of configuring and managing.

The playbook implements the following tasks:
- _TODO: In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
- ssh into Jump-Box-Provisioner
- start and attach your container
- /etc/ansible/roles directory and create elk playbook
- run elk playbook
- ssh into elk vm to be sure the server is up and running

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- _TODO: List the IP addresses of the machines you are monitoring_
10.1.0.4
10.0.0.9
10.0.0.10

We have installed the following Beats on these machines:
- _TODO: Specify which Beats you successfully installed_
filebeat
metricbeat

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._

-filebeat collects log files from specified files on machines (files from apache)
-metricbeat colects metrics/statistics (CPU uptime/usage)

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the _filebeat-playbook.yml file to _/etc/ansible/roles/files.
- Update the _filebeat.cfg_ file to include..._ELK private ip_ 
- Run the playbook, and navigate to _http://40.125.65.246:5601/app/kibana to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- Which file is the playbook? _filebeat-playbook.yml_ 

- Where do you copy it? _/etc/ansible/roles_

- Which file do you update to make Ansible run the playbook on a specific machine? _/etc/ansible/hosts_

- How do I specify which machine to install the ELK server on versus which to install Filebeat on? _in hosts file under webservers input the ip addresses of your VMs that you will install filebeat too and create one [elk] and input the IP of the vm elk is installed too or that you will install it too_

- Which URL do you navigate to in order to check that the ELK server is running? _http://20.38.173.69/_

- As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc.

_nano filebeat-playbook.yml to create playbook_ fill this out with your commands

_run playbook with ansible-playbook filebeat-playbook.yml_

_make sure you are in the correct directory where the playbook is located_


