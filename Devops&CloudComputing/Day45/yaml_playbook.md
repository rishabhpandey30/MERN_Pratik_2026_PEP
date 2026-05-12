# ANSIBLE YAML PLAYBOOKS CHEATSHEET
# ------------------------------------------------

# 1. Simple Ping Playbook
# Checks connectivity to all managed nodes

---
- name: Ping all servers
  hosts: all
  tasks:
    - name: Test connection
      ping:



# 2. Install Apache Web Server
# Installs Apache package on webservers

---
- name: Install Apache
  hosts: webservers
  become: yes

  tasks:
    - name: Install httpd package
      yum:
        name: httpd
        state: present



# 3. Start and Enable Apache Service
# Starts Apache and enables it on boot

---
- name: Start Apache Service
  hosts: webservers
  become: yes

  tasks:
    - name: Start and enable Apache
      service:
        name: httpd
        state: started
        enabled: yes



# 4. Copy File to Remote Servers
# Copies a local file to managed nodes

---
- name: Copy configuration file
  hosts: all

  tasks:
    - name: Copy file
      copy:
        src: index.html
        dest: /tmp/index.html



# 5. Create Directory
# Creates a directory if not present

---
- name: Create project directory
  hosts: all

  tasks:
    - name: Create directory
      file:
        path: /opt/project
        state: directory



# 6. Create User
# Adds a new Linux user

---
- name: Create DevOps user
  hosts: all
  become: yes

  tasks:
    - name: Add user
      user:
        name: devops
        state: present



# 7. Remove User
# Deletes an existing user

---
- name: Remove user
  hosts: all
  become: yes

  tasks:
    - name: Delete user
      user:
        name: devops
        state: absent



# 8. Install Multiple Packages
# Installs multiple software packages together

---
- name: Install packages
  hosts: all
  become: yes

  tasks:
    - name: Install packages
      yum:
        name:
          - git
          - wget
          - curl
        state: present



# 9. Run Linux Commands
# Executes shell commands on servers

---
- name: Check uptime
  hosts: all

  tasks:
    - name: Run uptime command
      command: uptime



# 10. Restart Service Using Handler
# Restarts Apache only if config changes

---
- name: Configure Apache
  hosts: webservers
  become: yes

  tasks:
    - name: Copy Apache config
      copy:
        src: httpd.conf
        dest: /etc/httpd/conf/httpd.conf
      notify: Restart Apache

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted



# 11. Use Variables
# Defines reusable variables in playbooks

---
- name: Variables Example
  hosts: all

  vars:
    package_name: git

  tasks:
    - name: Install package
      yum:
        name: "{{ package_name }}"
        state: present



# 12. Gather System Information
# Collects server facts automatically

---
- name: Gather facts
  hosts: all

  tasks:
    - name: Print hostname
      debug:
        var: ansible_hostname



# 13. Create File with Content
# Creates a file and writes content into it

---
- name: Create sample file
  hosts: all

  tasks:
    - name: Create file
      copy:
        dest: /tmp/demo.txt
        content: |
          Welcome to Ansible
          Automation is powerful.



# 14. Reboot Servers
# Restarts remote machines safely

---
- name: Reboot servers
  hosts: all
  become: yes

  tasks:
    - name: Reboot machine
      reboot:



# 15. Conditional Task Execution
# Runs task only on RedHat systems

---
- name: Conditional example
  hosts: all

  tasks:
    - name: Install Apache on RedHat
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
