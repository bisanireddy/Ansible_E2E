# Ansible_E2E

# First Step to check the ansible configurations are fine for windows based communication.

Location of config file is unser /etc/ansible/ansbile.cfg (case sensitive)

use nano /etc/ansbile/ansbile.cfg

# check the inventory file is created properly. the name should be exactlty matching with the name used in config file.
here you can specify your own variables for your windows group 


# check the powershell script ran on the windows to the Winrm configuration and ports are opened and also the username is correct and password using to connect is correct.


#now ypu can check the communication of controller node and windows host.
First do the manul ping if it succeed then work for ansbile ping

ansible all -m win_ping
![image](https://github.com/user-attachments/assets/499af472-b11d-4823-b985-c3563de25967)
