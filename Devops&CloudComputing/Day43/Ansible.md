# Ansible 
-----------------------------------------------------------------------
# Introduction to Configuration Management:
CM is the Process of maintaining computer systems, servers, and software in a desired, consistent state.

- Problem: Manually configuring 50 servers via SSH is time-consuming and prone to human error.

- Solution: Ansible automates this. You write the configuration once, and it applies it to all 50 servers simultaneously.
------------------------------------------------------------------------

Q.What is Ansible?

- Ansible is an open-source automation engine used for Configuration Management, Infrastructure Provisioning, and Application Deployment.

# Key Characteristics:
Agentless: No need to install any software (agents) on the client/target servers. It uses SSH for Linux and WinRM for Windows.

Idempotent: This is the most important feature. It means if you run the same script multiple times, it will only make changes if the current state doesn't match the desired state. If the work is already done, Ansible does nothing.

Declarative: You tell Ansible what you want (e.g., "Apache should be installed"), and Ansible figures out how to do it.
------------------------------------------------------------------------
# Ansible Architecture:

- Ansible works by connecting to your nodes and pushing out small programs, called Ansible Modules.

- Control Node: The machine where Ansible is installed.

- Managed Nodes: The target servers (nodes) that Ansible manages.

- Inventory: A file (usually hosts) that contains the list of IP addresses or hostnames of managed nodes.

- Modules: Pre-written scripts that Ansible executes (e.g., apt, yum, copy, service).