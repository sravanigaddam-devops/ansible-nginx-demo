# Deploying Web Application Using Ansible on Nginx - EC2

## Objective
To create an automation script to deploy a blogging web application (Sammy the Shark) on a remote Nginx server using Ansible. This is a Course-End Project for Configuration Management with Ansible and Terraform.

## Problem Statement
XYZ Pvt. Ltd. is a blogging platform where users create profiles and publish blogs. The application is ready and needs to be hosted on a remote server. As a DevOps Engineer, you are tasked to automate deployment on a remote Nginx server using Ansible - replacing manual SSH and configuration with an idempotent, reusable playbook.

## Industry Relevance
**Ansible** automates configuration management, application deployment and orchestration using simple YAML playbooks. It eliminates manual errors, enables infrastructure as code, and manages complex environments with agentless SSH architecture.

## Architecture

## Prerequisites
- AWS EC2 Ubuntu 22.04 (t2.micro) with Security Group: 22 (SSH/My IP), 80 (HTTP/0.0.0.0)
- Key pair `controller-key.pem` (chmod 400)
- Ansible 2.10+ on Control Node (`sudo apt install ansible -y`)
- SSH connectivity: `ansible -i inventory.yml all -m ping` -> pong

## Project Structure

## Tasks Completed

### Task 1: Inventory File
Defines remote server group `[web]` with `ansible_host`, `ansible_user=ubuntu`, `ansible_ssh_private_key_file` and `ansible_become=yes`.
```ini
[web]
sammy_server ansible_host=3.90.110.93 ansible_user=ubuntu ansible_ssh_private_key_file=/home/sravani08/.ssh/controller-key.pem ansible_ssh_common_args='-o StrictHostKeyChecking=no'

Task 2 & 5: Playbook Tasks
Playbook playbook.yml targets hosts: web with become: yes:

Update apt cache and install Nginx - apt: name=nginx update_cache=yes
Create document root - file: path=/var/www/sammy state=directory
Copy website files - copy: src=files/index.html dest=/var/www/sammy/index.html
Apply Nginx template - template: src=templates/nginx.conf.j2 dest=/etc/nginx/sites-available/sammy
Enable Nginx site - file: src=/etc/nginx/sites-available/sammy dest=/etc/nginx/sites-enabled/sammy state=link
Remove default site - file: path=/etc/nginx/sites-enabled/default state=absent
Allow tcp port 80 - ufw: rule=allow port=80 proto=tcp
Handler: Restart Nginx - triggers on config change

Task 3: Templates
templates/nginx.conf.j2 uses Jinja2 variables:

server {
    listen {{ nginx_port }};
    root {{ app_root }};
    index index.html;
    server_name _;
    location / { try_files $uri $uri/ =404; }
}

Task 4: Variables
Defined in playbook vars::

vars:
  app_root: /var/www/sammy
  nginx_port: 80

Task 6: Execution
ansible-playbook -i inventory.yml playbook.yml
# PLAY RECAP => ok=7 changed=5 failed=0 unreachable=0
curl http://3.90.110.93
# Browser: http://3.90.110.93 -> Sammy the Shark


How to Run Manually
git clone https://github.com/sravanigaddam-devops/ansible-nginx-demo.git
cd ansible-nginx-demo
chmod 400 ~/.ssh/controller-key.pem
ansible -i inventory.yml all -m ping
ansible-playbook -i inventory.yml --syntax-check playbook.yml
ansible-playbook -i inventory.yml playbook.yml

Troubleshooting
Permission denied (publickey) -> check chmod 400 and ansible_ssh_private_key_file path is /home/sravani08/.ssh/ not ~/
Unable to parse inventory -> use -i inventory.yml not -i inventory
Only Nginx default page -> ensure default site removed: file: path=/etc/nginx/sites-enabled/default state=absent
Indentation missing -> always create YAML with cat > file << 'EOF' (2-space indent, no tabs)

Tech Stack
Ansible
Nginx
AWS EC2 Ubuntu 22.04
YAML, Jinja2, UFW

