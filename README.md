# Ansible Playbook Deployment on AWS EC2 Infrastructure

A production-grade Infrastructure as Code (IaC) solution demonstrating automated infrastructure provisioning, configuration management, and application deployment using Ansible across AWS EC2 instances.

## 🎯 Project Overview

This project showcases **complete Ansible automation** for managing distributed infrastructure at scale. By automating Nginx installation, service configuration, and web content deployment across multiple EC2 instances, this project demonstrates real-world DevOps practices that replace manual SSH and repetitive server configuration with scalable, version-controlled automation.

**Key Achievement**: Automated full-stack provisioning and deployment across 3 managed nodes with zero manual intervention, ensuring repeatable and scalable infrastructure setup.

## ✨ Key Features

- **Multi-Node Infrastructure**: 1 control node + 3 managed nodes on AWS EC2
- **Inventory Management**: Organized servers into logical groups (server, prd)
- **SSH Security**: Secure key-based authentication via .pem files
- **Ansible Playbooks**: Automated orchestration for consistent deployments
- **Nginx Deployment**: Complete web server installation and configuration
- **Static Content Delivery**: Automated web page deployment
- **Idempotent Automation**: Safe, repeatable playbook execution
- **Infrastructure as Code**: Version-controlled, auditable infrastructure



## 🛠️ Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Infrastructure** | AWS EC2 | Compute instances |
| **Orchestration** | Ansible | Configuration management & automation |
| **Web Server** | Nginx | HTTP server for web content |
| **Authentication** | SSH (RSA Keys) | Secure node communication |
| **IaC Format** | YAML | Playbook definitions |
| **OS** | Linux (Ubuntu/Amazon Linux) | Managed node OS |

## 📊 Infrastructure Components

### Control Node Setup

**EC2 Instance Configuration**:
Instance Type: t2.micro (or t2.small for production)
OS: Ubuntu 20.04 LTS or Amazon Linux 2
Security Group: Allow SSH (22) from operator IP
Key Pair: ansible-control.pem

Installation:

Python 3.x

Ansible 2.10+

SSH client (pre-installed)



**Ansible Directory Structure**:
/home/ubuntu/ansible/
├── ansible.cfg # Ansible configuration
├── hosts # Inventory file
├── roles/ # Reusable roles (optional)
│ ├── nginx-install/
│ │ ├── tasks/
│ │ ├── templates/
│ │ └── handlers/
│ └── web-deploy/
├── playbooks/
│ ├── install-nginx.yml # Nginx installation
│ ├── deploy-website.yml # Website deployment
│ └── common-setup.yml # Common tasks
└── keys/
└── nodes.pem # SSH private key for nodes



### Inventory File Structure

**File**: `/etc/ansible/hosts` or `hosts`

[server]
server-1 ansible_host=10.0.1.100 ansible_user=ubuntu ansible_private_key_file=~/.ssh/nodes.pem
server-2 ansible_host=10.0.1.101 ansible_user=ubuntu ansible_private_key_file=~/.ssh/nodes.pem

[prd]
prd-1 ansible_host=10.0.1.102 ansible_user=ubuntu ansible_private_key_file=~/.ssh/nodes.pem

[all:vars]
ansible_python_interpreter=/usr/bin/python3



### SSH Connection Variables

**Ansible Connection Requirements**:
ansible_host: # EC2 instance IP address
ansible_user: # SSH user (ubuntu, ec2-user, etc.)
ansible_private_key_file: # Path to .pem SSH key
ansible_python_interpreter: # Python path on managed node
ansible_ssh_common_args: # Additional SSH options



## 📝 Playbook Examples

### Playbook 1: Nginx Installation

**File**: `playbooks/install-nginx.yml`

name: Install and start Nginx
hosts: server
become: yes
gather_facts: yes

tasks:

name: Update package manager
apt:
update_cache: yes
cache_valid_time: 3600
when: ansible_os_family == "Debian"

name: Install Nginx
package:
name: nginx
state: present

name: Start Nginx service
service:
name: nginx
state: started
enabled: yes

name: Verify Nginx is running
uri:
url: "http://{{ ansible_host }}:80"
status_code: 200
register: nginx_status
until: nginx_status.status == 200
retries: 3
delay: 5

name: Display Nginx status
debug:
msg: "✅ Nginx is running on {{ ansible_host }}"



**Execution**:
ansible-playbook playbooks/install-nginx.yml

Installs Nginx on server-1 and server-2 only


### Playbook 2: Website Deployment

**File**: `playbooks/deploy-website.yml`

name: Deploy static website
hosts: prd
become: yes

vars:
web_root: /var/www/html
site_name: "My Awesome Site"

tasks:

name: Install Nginx on production
package:
name: nginx
state: present

name: Start Nginx
service:
name: nginx
state: started
enabled: yes

name: Create web root directory
file:
path: "{{ web_root }}"
state: directory
owner: www-data
group: www-data
mode: '0755'

name: Deploy HTML file
copy:
content: |
<!DOCTYPE html>
<html>
<head>
<title>{{ site_name }}</title>
<style>
body { font-family: Arial, sans-serif; }
h1 { color: #333; }
</style>
</head>
<body>
<h1>Welcome to {{ site_name }}</h1>
<p>Deployed via Ansible from {{ inventory_hostname }}</p>
<p>Timestamp: {{ ansible_date_time.iso8601 }}</p>
</body>
</html>
dest: "{{ web_root }}/index.html"
owner: www-data
group: www-data
mode: '0644'

name: Verify website is accessible
uri:
url: "http://{{ ansible_host }}/"
status_code: 200
register: website_status

name: Display deployment status
debug:
msg: "✅ Website deployed successfully on {{ inventory_hostname }}"



**Execution**:
ansible-playbook playbooks/deploy-website.yml

Deploys website to prd-1 only


## 🔐 SSH Key Management

### Secure Key Transfer

**1. Generate SSH Key Pair (if not exists)**:
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible-nodes.pem -N ""



**2. Transfer .pem Key to Control Node**:
scp -i ~/.ssh/control-key.pem
~/.ssh/ansible-nodes.pem
ubuntu@control-node-ip:~/.ssh/



**3. Set Proper Permissions**:
chmod 600 ~/.ssh/ansible-nodes.pem



**4. Configure Ansible to Use Key**:
[nodes]
server-1 ansible_private_key_file=~/.ssh/ansible-nodes.pem



## 🚀 Playbook Execution

### Run All Playbooks
Run Nginx installation on all server nodes
ansible-playbook playbooks/install-nginx.yml

Run website deployment on production nodes
ansible-playbook playbooks/deploy-website.yml

Run both with verbosity
ansible-playbook playbooks/install-nginx.yml -v



### Dry Run (Check Mode)
Test without making changes
ansible-playbook playbooks/install-nginx.yml --check

Show what would change
ansible-playbook playbooks/install-nginx.yml --diff



### Run on Specific Hosts
Only on server-1
ansible-playbook playbooks/install-nginx.yml -l server-1

Only on prd group
ansible-playbook playbooks/deploy-website.yml -l prd



### Run Specific Tasks
Only run tasks tagged "deploy"
ansible-playbook playbooks/deploy-website.yml -t deploy

Skip tasks tagged "debug"
ansible-playbook playbooks/deploy-website.yml --skip-tags debug



## 📊 Execution Output Example

PLAY [Install and start Nginx] ****

TASK [Gathering Facts] ****
ok: [server-1]
ok: [server-2]

TASK [Update package manager] ****
changed: [server-1]
changed: [server-2]

TASK [Install Nginx] ****
changed: [server-1]
changed: [server-2]

TASK [Start Nginx service] ****
changed: [server-1]
changed: [server-2]

TASK [Verify Nginx is running] ****
ok: [server-1]
ok: [server-2]

PLAY RECAP ****
server-1: ok=5 changed=4 unreachable=0 failed=0
server-2: ok=5 changed=4 unreachable=0 failed=0



## 🎓 Key Learning Outcomes

Through this project, I demonstrated expertise in:

✅ **Ansible Fundamentals**: Playbooks, tasks, handlers, and variables  
✅ **Inventory Management**: Host grouping and SSH variables  
✅ **SSH Security**: Key-based authentication and .pem file management  
✅ **Idempotent Operations**: Safe, repeatable automation  
✅ **Web Server Deployment**: Nginx installation and configuration  
✅ **Content Delivery**: Automated website deployment  
✅ **Infrastructure as Code**: Version-controlled infrastructure  
✅ **AWS EC2 Integration**: Multi-instance orchestration  
✅ **Automation Best Practices**: Error handling and validation  

## 📈 Automation Benefits

| Before (Manual SSH) | After (Ansible) |
|---|---|
| SSH into each server individually | Single command deploys to all servers |
| Manually install packages | Automated package installation |
| Manual configuration | Consistent configuration across all nodes |
| Prone to human error | Repeatable, error-proof execution |
| No audit trail | Full version control and history |
| Hours for deployment | Minutes for deployment |
| Difficult to scale | Scales to 100+ servers effortlessly |

## 🔄 Idempotency Guarantee

**Ansible ensures idempotency**: Running the same playbook multiple times produces the same result without side effects.

First run: Installs Nginx
ansible-playbook playbooks/install-nginx.yml

Changed: 3 tasks
Second run: Nothing changes (already installed)
ansible-playbook playbooks/install-nginx.yml

Changed: 0 tasks (all "ok")
Third run: Same as second run
ansible-playbook playbooks/install-nginx.yml

Changed: 0 tasks


## 🚀 Real-World Use Cases

- **Web Server Deployment**: Deploy Nginx across multiple servers instantly
- **Application Updates**: Push new versions to all servers simultaneously
- **Configuration Management**: Manage thousands of servers consistently
- **Environment Provisioning**: Create identical dev/staging/prod environments
- **Disaster Recovery**: Quickly rebuild infrastructure from playbooks

## 🔒 Security Best Practices

✅ **SSH Key Management**: Secure .pem storage and permissions  
✅ **Privilege Escalation**: Using `become` for elevated tasks  
✅ **Ansible Vault**: Encrypt sensitive variables  
✅ **Minimal Permissions**: Least-privilege access  
✅ **Audit Logging**: Track all deployments  

## 📄 License

Apache 2.0


**Project Timeline**: July 2025  
**Technologies**: Ansible · AWS EC2 · Nginx · SSH · YAML · Infrastructure as Code  
**Key Achievement**: Automated full-stack provisioning and deployment without manual intervention  
**Real-World Impact**: Eliminated repetitive manual work, enabling scalable infrastructure management
