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
???

copied Bens' ansible-playbooks-zos to /home/neale/ansible
No luck....
park for now

```
[neale@localhost ansible]$ pwd
/home/neale/ansible
[neale@localhost ansible]$ ls -al
total 112
drwxr-xr-x.  3 neale neale  4096 Aug 24 16:26 .
drwx------. 21 neale neale  4096 Aug 24 16:25 ..
-rw-r--r--.  1 neale neale  3423 Aug 24 16:26 base_auto_mount.yml
-rw-r--r--.  1 neale neale  8670 Aug 24 16:26 base_COBOL.yml
-rw-r--r--.  1 neale neale  1372 Aug 24 16:26 base_ispplib.yml
-rw-r--r--.  1 neale neale  6644 Aug 24 16:26 base_jes.yml
-rw-r--r--.  1 neale neale  3444 Aug 24 16:26 base_jobcard.yml
-rw-r--r--.  1 neale neale 10273 Aug 24 16:26 base_parmlib.yml
-rw-r--r--.  1 neale neale  3714 Aug 24 16:26 base_proclib.yml
-rw-r--r--.  1 neale neale  4564 Aug 24 16:26 base_rmf.yml
-rw-r--r--.  1 neale neale  7182 Aug 24 16:26 cics-egui-CSD.jcl
-rw-r--r--.  1 neale neale  1300 Aug 24 16:26 cics-egui-EXMPCAT.txt
-rw-r--r--.  1 neale neale   286 Aug 24 16:26 cics-egui-EXMPCONF.txt
-rw-r--r--.  1 neale neale 11199 Aug 24 16:26 cics-egui.yml
drwxr-xr-x.  3 neale neale    45 Aug 24 16:27 inventories
-rw-r--r--.  1 neale neale    11 Aug 24 16:26 nealetest.yml
-rw-r--r--.  1 neale neale  3345 Aug 24 16:26 smpe_zfs.yml
-rw-r--r--.  1 neale neale  1420 Aug 24 16:26 vtp.yml
-rw-r--r--.  1 neale neale  3920 Aug 24 16:26 z25a_housekeeping.yml
-rw-r--r--.  1 neale neale  1758 Aug 24 16:26 zos_ping.yml
```

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
