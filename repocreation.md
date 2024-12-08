#first install Git 
sudo yum update -y
sudo yum install git -y

Once it installed, check verison to validate

git --version

![image](https://github.com/user-attachments/assets/5d4cae9c-3dcf-4672-964c-f0dfd08fa2e5)


clone your repo

git clone https://github.com/bisanireddy/Ansible_E2E.git

the repo folername will be automatically created.

change the current directory to the repo directory and open the VScode

code .


If you are a super user, exit and run as normal user

![image](https://github.com/user-attachments/assets/2f7ed5c6-3a7d-4a46-a23c-887fa6bf2527)

#install the nessesary extensions
To enhance your experience with Ansible on VS Code:

Open the Extensions view (Ctrl+Shift+X).
Search for and install the following extensions:
Ansible (by Red Hat)
YAML (by Red Hat)
Python (if your playbooks include Python scripts).
