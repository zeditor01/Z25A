# Ansible Setup

I lost the docco for setup. But it was something like

* github repository contains ansible scripts
* use certicate for VSCode github client
* use certificate for CentOS github client
* install ansible on CentOS

## Ansible Playbooks on PC
C:\Users\neale\OneDrive\Github\ansible-playbooks\ansible-playbooks-zos
VSCODE : SYNC with Git 

## Ansible Core on NUC i3 (192.168.1.199)
/home/neale/Github/ansible-playbooks/ansible-playbooks-zos
Sync with github 

Cd /home/neale/Github/ansible-playbooks
Git pull

Invoke a playbook on CentOS 

Cd /home/neale/Github/ansible-playbooks/ansible-playbooks-zos
ansible_playbook zos_ping.yml 

## Ansible Version

[neale@localhost ~]$ ansible --version
ansible [core 2.13.1]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/neale/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /home/neale/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.9.10 (main, Feb  9 2022, 00:00:00) [GCC 11.2.1 20220127 (Red Hat 11.2.1-9)]
  jinja version = 3.1.2
  libyaml = True
![image](https://user-images.githubusercontent.com/98128779/186333663-24653524-0b6b-42f4-837a-798b674d94dc.png)
