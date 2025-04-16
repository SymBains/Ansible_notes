# Infrastructure as Code

**Infrastructure as Code (IaC)** is a practice that uses code to provision and manage computing infrastructure, replacing manual processes. It allows developers to define infrastructure in configuration files, enabling automation, consistency, and scalability.

## Key Benefits

- **Automation**: Rapidly set up and manage environments.
- **Environment Duplication**: Reuse the same configuration to replicate environments across systems.
- **Error Reduction**: Minimizes manual mistakes and supports quick rollbacks.
- **Version Control**: Easily track changes and iterate using source control tools.

## How IaC Works

- Infrastructure is defined using configuration files similar to application code.
- IaC files can be written in common programming languages (e.g., Python, Java).
- Managed in IDEs with built-in error checking and source control integration.

## IaC in DevOps

- Integrated into **CI/CD pipelines** for automatic provisioning during builds/releases.
- Ensures **consistent configurations** across environments.
- Enhances collaboration between developers and IT operations.

## Configuration Management

**Configuration management** is the process of **systematically handling changes** to ensure a system’s **consistency, stability, and performance** over time.

In simple terms, it’s about:

- **Setting up** systems (like servers or software environments)
- **Keeping them in the desired state**
- **Ensuring consistency** even after updates or changes

## Key Concepts

- **Desired State**: What the system *should* look like  
  (e.g., "Apache should be installed", "Firewall should allow port 80")
  
- **Automation**: Use tools like **Ansible**, **Puppet**, or **Chef** to apply and maintain configurations automatically

- **Consistency**: Ensuring all environments (dev, test, production) are set up the same way

- **Version Control**: Configuration files are stored in Git to track, review, and roll back changes when needed

## What is Orchestration?

**Orchestration** is the automated arrangement, coordination, and management of complex systems, services, and processes.

Think of it like a **conductor leading an orchestra**—ensuring each instrument (or task) starts at the right time, in the right order, and works in harmony with others.

## Key Points:

- It’s about **automating multi-step workflows**
- Manages **dependencies** and **execution order**
- Often involves **multiple systems or services**

### Example:

You want to:

1. Create virtual machines  
2. Install software on them  
3. Start services in a specific order  
4. Notify another system when complete

Instead of doing this manually or through separate scripts, orchestration tools automate the entire flow.

## What is Ansible?

**Ansible** is an open-source **automation tool** used for:

- **Configuration management**
- **Application deployment**
- **Task automation**
- **Orchestration**

## Why Use Ansible?

- **Agentless**: No software needs to be installed on target machines
- Uses **SSH** to communicate with remote systems
- Configuration written in **YAML** (called *Playbooks*)
- Easy to read and maintain

### What Can Ansible Do?

- Install software packages (e.g., `nginx`, `docker`)
- Configure system settings (e.g., firewall, user permissions)
- Deploy applications across multiple servers
- Automate repetitive tasks (e.g., updates, service restarts)
- Orchestrate complex operations (e.g., rolling updates)

![Ansible](Images/ansible.png)

# Setting Up Ansible on Ubuntu EC2 Instances

This guide walks you through configuring **Ansible** with one **controller node** and one **target node**, both hosted on Ubuntu EC2 instances.

## Prerequisites

- Two **Ubuntu EC2 instances**:
  - One as the **Controller** (where Ansible will run)
  - One as the **Target Node** (which Ansible will manage)
- The **.pem SSH key** for your EC2 instances

## Set Up the Controller Node

### Update and Upgrade Packages
```bash
sudo apt update -y
sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y
```

### Add Ansible Repository and Install
```bash
sudo apt-add-repository ppa:ansible/ansible
sudo DEBIAN_FRONTEND=noninteractive apt install ansible -y
```

### Verify Installation
```bash
ansible --version
```

### Navigate to the Ansible Configuration Directory
```bash
cd /etc/ansible
ls
```

### Check Hidden Files and SSH Directory
```bash
cd ~
ls -a
cd .ssh
```

### Create the PEM File
Paste your private key (.pem) into a new file:

```bash
sudo nano tech503-symron-aws-key.pem
```

In a new terminal, copy the private key content from your local machine and paste it into the .pem file on the controller.

### Test SSH Connection to Target Node

From the controller:

```bash
ssh -i ~/.ssh/tech503-symron-aws-key.pem ubuntu@<TARGET_NODE_PUBLIC_IP>
```

If successful, type:
```bash
exit
```

### Configure the Ansible Inventory
Edit the hosts File
```bash
cd /etc/ansible
sudo nano hosts
```

Add the following at the bottom:
```bash
[web]
ec2-instance ansible_host=<TARGET_NODE_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/tech503-symron-aws-key.pem
```

```
sudo chmod 400 /home/ubuntu/.ssh/tech503-symron-aws-key.pem
```

### Test Ansible Connection
Run a ping module to test the setup:
```bash
sudo ansible all -m ping
```

### Update ansible.cfg (Optional but Recommended)
To avoid Python interpreter warnings:
```bash
sudo nano ansible.cfg
```

Add this to the bottom:
```bash
[defaults]
interpreter_python = auto_silent
host_key_checking = False
```

## Use Ad-Hoc Command to Copy PEM File

```bash
sudo ansible web -m copy -a "src=/home/ubuntu/.ssh/tech503-symron-aws-key.pem dest=/home/ubuntu/.ssh/tech503-symron-aws-key.pem mode=0600 owner=ubuntu group=ubuntu"
```

Breakdown of Command:
- web: The inventory group name (from /etc/ansible/hosts)
- -m copy: Use the copy module
- -a: Provide arguments to the module:
- src: Local path on the controller
- dest: Target path on the target node
- mode=0600: Restrict permissions for security
- owner=ubuntu, group=ubuntu: Ensures correct file ownership

## Deploy NGINX with Ansible Ad-Hoc Commands
```bash
sudo ansible web -m apt -a "update_cache=yes" --become #Update APT Cache on Target Node
sudo ansible web -m apt -a "name=nginx state=present" --become #Install NGINX on Target Node, --become  is required to escalate privileges (like using sudo)
sudo ansible web -m service -a "name=nginx state=started enabled=yes" --become # NGINX is Started and Enabled
sudo ansible web -m command -a "systemctl status nginx" 
```


useful commands
Use the ansible-inventory command to see the current inventory

Try these handy switches:
--help
--list: display hosts in a list
--graph: display hosts in a tree view

ansible web -a "uname -a"           will show the version of linux on all the target nodes

sudo apt update using ansible:
 sudo ansible web -m ansible.builtin.apt -a "update_cache=yes" --become

sudo apt upgrade using ansible:
sudo ansible web -m ansible.builtin.apt -a "upgrade=dist" --become




