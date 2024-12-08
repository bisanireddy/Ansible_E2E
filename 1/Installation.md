![image](https://github.com/user-attachments/assets/3a7db46e-9775-4af7-8673-cac209b5dce6)Before

![image](https://github.com/user-attachments/assets/ab9b46c2-5470-4b5f-b34d-4390d8386de9)


After

![image](https://github.com/user-attachments/assets/1098b455-9ef8-4c4e-b634-ebc5fac0266f)


![image](https://github.com/user-attachments/assets/c625c3ba-079e-482e-ae66-1590fb37c5f3)

![image](https://github.com/user-attachments/assets/56385336-8bda-4048-b338-a189d8414ffe)

![image](https://github.com/user-attachments/assets/dfec2b5c-8504-4ff1-86a9-24b3c3293441)


![image](https://github.com/user-attachments/assets/f4a176ae-7eda-45db-8cf4-2f32e831c8f8)




example of 

- name: Install Chocolatey
  win_shell: |
    Set-ExecutionPolicy Bypass -Scope Process -Force; `
    [System.Net.ServicePointManager]::SecurityProtocol = `
    [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; `
    iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))



#using win_chocolatery is failed
![image](https://github.com/user-attachments/assets/6b3db117-a051-4192-8a02-f79193277ba8)


#Solution
ansible-galaxy collection install community.windows

![image](https://github.com/user-attachments/assets/c3058e35-7fe6-48ed-96ac-8ad48560bafc)

![image](https://github.com/user-attachments/assets/dc4abe14-a115-48dc-bfba-cf4722f8a3f7)


updating ansible

ansible --version
pip install --upgrade ansible
![image](https://github.com/user-attachments/assets/34eb316f-2bc9-446f-82d0-40e59158c666)


Before

![image](https://github.com/user-attachments/assets/033b9204-c6a6-425c-b2e4-afda33f1fe85)

![image](https://github.com/user-attachments/assets/dfd1c493-494f-449e-9081-1be9712ae5c6)

sudo dnf install ansible


Installing windows modules
![image](https://github.com/user-attachments/assets/93634326-da1e-4e65-8b1e-776ff56534dd)

Repo is not configured\

![image](https://github.com/user-attachments/assets/46f6709a-4bcc-4a50-8f5f-29fefa7fb096)



Check all the modules installed
ansible-galaxy collection list

![image](https://github.com/user-attachments/assets/5d861e35-83c0-4cb1-9471-15bbf9801f90)


![image](https://github.com/user-attachments/assets/094469c3-40e6-4438-957f-5ab68dfb2193)

![image](https://github.com/user-attachments/assets/7120bc9e-63c8-4574-afa1-4690ec9daa3e)

