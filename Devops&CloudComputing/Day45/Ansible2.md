# Advanced Components: Handlers, Roles, and Templates
# -------------------------------------------------------------------------
# Tasks & Handlers:
Tasks: The individual actions defined in a playbook that call a specific module.

- Handlers: Special tasks that only run when "notified" by another task. 
- They are typically used to restart a service only if a configuration file actually changed.
- Tasks :Individual actions executed on managed nodes.

Ex:-
```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present
    notify: restart nginx

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```
# Key point:
Each task uses a module
Executed sequentially (top → bottom)
If a task fails, playbook stops unless handled

# Roles:
Roles provide a way to package and organize automation into a standardized directory structure (e.g., /tasks, /vars, /templates).
This makes your code modular, reusable, and easy to share.

Ex:-
```bash
  roles/
  webserver/
    tasks/
    handlers/
    templates/
    vars/
    defaults/
    files/
    meta/
```
- This structure allows you to organize your automation code logically and reuse it across different playbooks or projects.

# Jinja2 Templates:
- Jinja2 templates allow dynamic configuration files using variables.
- Instead of hardcoding values, you use variables (e.g., {{ http_port }}) that Ansible fills in specifically for each managed node during  execution.

Ex:-File Example (nginx.conf.j2)
```nginx
server {
    listen {{ port }};
    server_name {{ domain }};
} 
```
# Playbook Usage:
- name: Deploy config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf

- This playbook task uses the "template" module to deploy the nginx.conf.j2 template to the target servers, replacing the {{ port }} and {{ domain }} variables with actual values defined in the playbook or inventory.

# Key Points:
Uses {{ variables }}
Enables dynamic configs
Common in system configuration files
------------------------------------------------------------------------
# Example Playbook:
```yaml 
---
- name: Setup Web Server
  hosts: webservers
  become: yes
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present

    - name: Copy Configuration File
      template:
        src: templates/httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf
      notify: Restart Apache

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted
```
In this example, the playbook sets up a web server by installing Apache and copying a configuration file. 
If the configuration file is changed, it triggers a handler to restart the Apache service.
------------------------------------------------------------------------
# Ansible Workflow:
1. Write an Inventory: Create an inventory file that lists your managed nodes and groups them as needed.
2. Create a Playbook: Write a playbook that defines the tasks you want to perform on the managed nodes.
3. Run the Playbook: Use the ansible-playbook command to execute the playbook against the inventory. 
   Ansible will connect to the managed nodes and perform the tasks defined in the playbook.
4. Review Results: Ansible will provide output showing which tasks were executed, which were changed, and if there were any errors.
# -----------------------------------------------------------------------
# ⚡Ad-hoc Commands:
- Ad-hoc commands are one-line Ansible commands used to perform quick tasks without writing a playbook.
- They are useful for simple operations like checking connectivity, gathering information, or running a command on multiple servers.
- Ad-hoc commands use the ansible command followed by the target hosts, module, and arguments.

Ex:- To check connectivity to all servers, you can run:
 
  ```bash
ansible all -m ping
```
- This command uses the "ping" module to test SSH connectivity to all hosts in the inventory.

Ex:Ping all hosts:
```bash
ansible all -m ping
``` 
- This command checks connectivity to all managed nodes using the "ping" module.

Ex:Install Apache on webservers:
```bash
ansible webservers -m yum -a "name=httpd state=present" --become
```
- This command installs the Apache package on all hosts in the "webservers" group using the "yum" module with elevated privileges.

Ex:Check disk usage:
```bash
ansible all -a "df -h"
```
- This command runs the "df -h" shell command on all managed nodes to check disk usage.

Ex:Gather system information:
```bash 
ansible all -m setup
```
- This command collects detailed system information (facts) from all managed nodes using the "setup" module.

