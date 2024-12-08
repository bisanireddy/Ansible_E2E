config file issues are solved by below

Option 2: Permanent Configuration
To make the ANSIBLE_CONFIG variable permanent:

Add the following line to your shell configuration file (~/.bashrc or ~/.bash_profile for Bash users):

bash
Copy code
export ANSIBLE_CONFIG=/etc/ansible/Ansible_E2E/ansible-windows-automation/ansible.cfg
Apply the changes:

bash
Copy code
source ~/.bashrc
Now, every time you log in, the ANSIBLE_CONFIG variable will point to your ansible.cfg.



forcing the playbook to use cfg file
ANSIBLE_CONFIG=/etc/ansible/Ansible_E2E/ansible-windows-automation/ansible.cfg ansible-playbook playbooks/setup_chocolatey.yml

Get all the collection list
ansible-galaxy collection list


reading the modules available in collectios
ansible-galaxy collection list
ansible-doc -l | grep 'community.windows'
