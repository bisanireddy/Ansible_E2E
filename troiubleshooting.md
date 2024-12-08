check the roles
ansible-galaxy list --roles-path=./roles


ansible-playbook playbooks/setup_chocolatey.yml -vvv
![image](https://github.com/user-attachments/assets/37fab517-1249-4f9f-b6f8-9cd2c2686fd4)

Option 1: Modify ansible.cfg
Add the following line to your ansible.cfg file under the [defaults] section:

[defaults]
# Other configurations
ansible_winrm_server_cert_validation = ignore

all:
  hosts:
    win_host1:
      ansible_connection: winrm
      ansible_user: Administrator
      ansible_password: yourpassword
      ansible_port: 5985
      ansible_winrm_transport: basic
      ansible_winrm_server_cert_validation: ignore  # Optional for HTTP



Option 2: Modify the Playbook
Alternatively, you can disable SSL verification directly in your playbook by using the winrm_server_cert_validation parameter:

- name: Install Chocolatey
  hosts: all
  tasks:
    - name: Install Chocolatey
      win_shell: |
        Set-ExecutionPolicy Bypass -Scope Process -Force;
        iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
      vars:
        ansible_winrm_server_cert_validation: ignore


installation succesful
![image](https://github.com/user-attachments/assets/1184dee5-f62b-47c2-b392-54f40166e4d9)
