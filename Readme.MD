## BEGIN ANSIBLE DEVELOPMENT


### Step 1 -  Create a directory called **ansible-config-mgt**

- Within **ansible-config-mgt** directory create a directory and name it **playbooks** – it - will be used to store all your playbook files.
- Create a directory and name it **inventory** – it will be used to keep your hosts organised.
- Within the playbooks folder, create your first playbook, and name it **common.yml**
- Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) for example : dev.yml, staging.yl, uat.yml, and prod.yml respectively.

### Step 2 – Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

Save below inventory structure in the inventory/dev file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:

To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, please see this video:

For Windows users – [ssh-agent on windows](https://youtu.be/OplGrY74qog)
For Linux users – [ssh-agent on linux](https://youtu.be/OplGrY74qog)

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```
Confirm the key has been added with the command below, you should see the name of your key
```
ssh-add -l
```

Now, ssh into your Jenkins-Ansible server using ssh-agent
```
ssh -A ubuntu@public-ip
```
Also notice, that your Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.

Update your inventory/dev.yml file with this snippet of code

```
[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'
```

### Step 3 – Create a Common Playbook

It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in **inventory/dev**.

In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your **playbooks/common.yml** file with following code:

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

```

Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

### Step 4 – Run Ansible Playbook

```
cd ansible-config-mgt
ansible-playbook -i inventory/dev.yml playbooks/common.yml
```
You can go to each of the servers and check if wireshark has been installed by running which wireshark or wireshark --version
