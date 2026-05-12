# ANSIBLE AD-HOC COMMANDS
# ----------------------------------------------------------

# 1. Ping all managed nodes
# Checks connectivity and SSH access to all servers
ansible all -m ping


# 2. Ping a specific group
# Verifies connection only for webservers group
ansible webservers -m ping


# 3. Gather system information
# Collects facts like IP, OS, RAM, hostname, etc.
ansible all -m setup


# 4. Run Linux commands
# Executes shell commands on remote servers
ansible all -a "uptime"


# 5. Check disk usage
# Displays filesystem disk usage
ansible all -a "df -h"


# 6. Check memory usage
# Shows RAM usage on remote systems
ansible all -a "free -m"


# 7. Create a file
# Creates an empty file on remote servers
ansible all -m file -a "path=/tmp/demo.txt state=touch"


# 8. Create a directory
# Creates a directory if it doesn't exist
ansible all -m file -a "path=/opt/project state=directory"


# 9. Delete a file
# Removes a file from remote servers
ansible all -m file -a "path=/tmp/demo.txt state=absent"


# 10. Copy a file
# Copies a local file to remote systems
ansible all -m copy -a "src=index.html dest=/tmp/index.html"


# 11. Install a package (RHEL/CentOS)
# Installs software using yum
ansible all -m yum -a "name=httpd state=present" --become


# 12. Remove a package
# Uninstalls software packages
ansible all -m yum -a "name=httpd state=absent" --become


# 13. Start a service
# Starts a service on remote servers
ansible all -m service -a "name=httpd state=started" --become


# 14. Stop a service
# Stops a running service
ansible all -m service -a "name=httpd state=stopped" --become


# 15. Restart a service
# Restarts a service after config changes
ansible all -m service -a "name=httpd state=restarted" --become


# 16. Enable a service
# Ensures service starts automatically on boot
ansible all -m service -a "name=httpd enabled=yes" --become


# 17. Create a user
# Adds a new user account
ansible all -m user -a "name=devops password='password'" --become


# 18. Delete a user
# Removes a user account
ansible all -m user -a "name=devops state=absent" --become


# 19. Reboot servers
# Restarts remote machines safely
ansible all -m reboot --become


# 20. Check running processes
# Displays active processes
ansible all -a "ps -ef | grep nginx"


# 21. Update all packages
# Updates all installed packages to latest versions
ansible all -m yum -a "name=* state=latest" --become


# 22. Fetch remote files
# Downloads files from remote servers to control node
ansible all -m fetch -a "src=/var/log/messages dest=/tmp/"


# 23. Change file permissions
# Modifies permissions of files/directories
ansible all -m file -a "path=/tmp/demo.txt mode=0777"


# 24. Execute shell scripts
# Runs shell scripts on managed nodes
ansible all -m shell -a "/tmp/script.sh"


# 25. Check hostname
# Displays hostname of all servers
ansible all -a "hostname"


# 26. Shutdown servers
# Powers off remote systems
ansible all -a "shutdown -h now" --become


# 27. Check OS version
# Displays operating system details
ansible all -a "cat /etc/os-release"


# 28. Check logged-in users
# Shows currently logged-in users
ansible all -a "who"


# 29. Check CPU information
# Displays processor details
ansible all -a "lscpu"


# 30. Run command as sudo
# Executes commands with elevated privileges
ansible all -b -a "systemctl status httpd"