# Ansible Basics

>Ansible is an open-source automation tool for IT tasks such as 
> - configuration management
> - application deployment
> - orchestration

##  What is Ansible?
 Ansible is a powerful automation tool used for:
 - Configuration management: Automate server setup, manage services, and configurations.
 - Application deployment: Deploy and manage software applications.
 - Orchestration: Coordinate complex, multi-step workflows, such as CI/CD pipelines.

Ansible operates by connecting over SSH, and it doesn’t require any agent to be installed on the target nodes. This makes it lightweight and simple to use.

## Key Components of Ansible
Ansible consists of several key components:

- Inventory: A list of servers (hosts) Ansible manages, which can be in the form of an inventory file or dynamic inventory.
- Playbook: A YAML file that defines a series of tasks to be executed on the hosts.
- Module: Small programs that Ansible runs to perform system changes (e.g., managing files, services).
- Task: The individual unit of action in Ansible, such as installing a package or starting a service.
- Role: A way to organize tasks, handlers, and variables in a structured way for reuse and sharing.
- Handlers: Tasks triggered by changes, usually at the end of playbook execution.
- Variables: Store data dynamically, allowing reusability and customization.
- Templates: Generate dynamic files using the Jinja2 templating engine.
## Setting Up Ansible
Installation
For most systems (e.g., Ubuntu, Debian, CentOS), installing Ansible is straightforward:

On Ubuntu/Debian:
```
sudo apt update
sudo apt install ansible -y
```

Verify the installation:

> ansible --version
Basic Directory Structure
After installing Ansible, the directory structure for your projects might look like this:
```
inventory/       # Contains the list of hosts
playbooks/       # Directory for storing your playbooks
roles/           # Directory for Ansible roles
ansible.cfg      # Configuration file
```
Inventory File  
Create a simple inventory file:  
inventory.ini 
```

[aws_servers]
54.253.108.23 ansible_user=ec2-user
3.27.68.64 ansible_user=ec2-user
3.24.138.192 ansible_user=ec2-user
3.24.110.18 ansible_user=ec2-user

[all:vars]
ansible_ssh_private_key_file=MyKeyPair.pem

```

## Basic Ansible Concepts
### Ad-Hoc Commands
Ansible allows you to run commands on remote hosts without writing a playbook. For example, to check connectivity, run:
> ansible all -i inventory -m ping
To install nginx on all web servers:
```
ansible web -i inventory -m apt -a "name=nginx state=present" --become
-i: Specifies the inventory file.
-m: Specifies the module to use (ping, apt, etc.).
-a: Provides arguments for the module.
--become: Executes the task with sudo or privilege escalation.
```
## Writing Playbooks
A playbook is a YAML file containing one or more “plays.” Each play defines a set of tasks to be executed on a set of hosts.

Simple Playbook Example:
```
---
- name: Install and configure nginx
  hosts: web
  become: true
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    - name: Start nginx
      service:
        name: nginx
        state: started
```        
This playbook installs and starts NGINX on all web hosts defined in the inventory.

####Task Structure
Each task typically includes:

- name: A description of the task.
- module: The Ansible module to use (apt, service, etc.).
- arguments: Options for the module.
- Conditionals and Loops

``` 
- name: Install nginx on Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_distribution == "Ubuntu"
```
- Loops allow you to repeat a task for a list of items:
```
- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - git
```    
## Roles in Ansible
Roles allow you to organize tasks, variables, and handlers in a reusable structure. Roles are typically stored in the roles/ directory.

Basic Role Structure
```
roles/
  myrole/
    tasks/
      main.yml
    handlers/
      main.yml
    templates/
    files/
    vars/
      main.yml
```
To create a role:

> ansible-galaxy init myrole
You can call a role in a playbook:
```
---
- hosts: web
  roles:
    - myrole
```

## Ansible Vault: Securing Secrets
Ansible Vault is used to encrypt sensitive data such as passwords or API keys in your playbooks. This ensures that confidential information isn’t exposed in your version control system.

Encrypt a file:
> ansible-vault encrypt secrets.yml
Decrypt a file:
> ansible-vault decrypt secrets.yml
Edit an encrypted file:
> ansible-vault edit secrets.yml
Using Vault in Playbooks:
After creating a secrets.yml file, use it in a playbook:
```
---
- hosts: all
  vars_files:
    - secrets.yml
  tasks:
    - name: Print the secret
      debug:
        msg: "{{ secret_variable }}"
```        
To run a playbook with Vault:

> ansible-playbook playbook.yml --ask-vault-pass
This is especially useful when you need to manage sensitive data securely across multiple environments.

## Advanced Topics in Ansible
> Dynamic Inventory
In some cases, you may not have static IP addresses or hostnames. Dynamic inventory allows you to pull inventory data from cloud providers like AWS, GCP, or Azure dynamically.  

To use dynamic inventory:  

Install the necessary inventory plugin (e.g., AWS EC2):

>pip install boto boto3
Configure the dynamic inventory script (ec2.py) in your ansible.cfg file:

> [inventory] enable_plugins = aws_ec2
Run playbooks against dynamic inventory:
```
ansible-playbook playbook.yml -i ec2.py
Delegation and Parallelism
Delegation: Run a task on one host but apply it to another. For example, creating a user on a web server and also adding it to the load balancer:
tasks:   - name: Configure load balancer     command: /usr/local/bin/configure-lb     delegate_to: lb.example.com
Parallelism: Control the number of hosts Ansible runs against in parallel using the -f option:
ansible-playbook playbook.yml -f 10
```
Callbacks and Notifications
Ansible allows you to integrate with external systems, sending notifications on task completion (e.g., to Slack or email). Callback plugins make this possible.

Enable a callback plugin in ansible.cfg:

[defaults]
callbacks_enabled = timer, mail
You can also develop custom callback plugins to integrate with other services.

## Ansible Galaxy and Reusable Content
```
Ansible Galaxy is the official hub for sharing Ansible roles, making it easier to reuse configurations.

Using Galaxy Roles:  
Install a role from Ansible Galaxy:

ansible-galaxy install geerlingguy.nginx
Use the installed role in your playbook:

--- - hosts: all   roles:     - geerlingguy.nginx
Publishing Roles to Galaxy:
To share a role with the community, follow these steps:

Create an account on Ansible Galaxy.
Initialize a role:
ansible-galaxy init myrole
 Add metadata and documentation, then publish your role using the GitHub integration.
 ```


## Ansible Tower (AWX): Enterprise Automation
Ansible Tower (or its open-source counterpart AWX) is a web-based solution for scaling and managing Ansible automation in enterprise environments.

>Key features include:

- Centralized logging and job tracking.
- RBAC (Role-Based Access Control) to manage who can run what.
- Playbook scheduling and workflow automation.
- Graphical interface for monitoring and managing playbook runs.
- Setting Up AWX:
- AWX can be installed via Docker:

- git clone https://github.com/ansible/awx.git
- cd awx/installer
- ansible-playbook -i inventory install.yml
- After installation, you can access AWX via a browser to manage your Ansible workflows.



### Referece
https://medium.com/@shivam2003/ansible-a-complete-guide-from-basics-to-advanced-ffd1bf74322f
https://www.tutorialspoint.com/ansible/ansible_roles.htm
