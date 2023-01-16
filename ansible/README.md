## [Ansible](https://www.ansible.com/) 
Ansible is an open source community project sponsored by Red Hat, it's the simplest way to automate IT.

### [Install Ansible](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04)
- `sudo apt install ansible`

### Directories
- [Inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)

### Playbook
#### [Roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)
Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure

By default Ansible will look in each directory within a role for a main.yml file for relevant content (also main.yaml and main):

- `tasks/main.yml` - the main list of tasks that the role executes.

- `handlers/main.yml` - handlers, which may be used within or outside this role.

- `library/my_module.py` - modules, which may be used within this role (see Embedding modules and plugins in roles for more information).

- `defaults/main.yml` - default variables for the role (see Using Variables for more information). These variables have the lowest priority of any variables available, and can be easily overridden by any other variable, including inventory variables.

- `vars/main.yml` - other variables for the role (see Using Variables for more information).

- `files/main.yml` - files that the role deploys.

- `templates/main.yml` - templates that the role deploys.

- `meta/main.yml` - metadata for the role, including role dependencies.

### How to use Ansible 

1. Install ansible and require package use pip(3):
```
pip3 install -r requirements.txt
```
![](https://i.imgur.com/BiPnIcB.png)

2. Use playbook
- Update `inventory/inventory.yaml` with the list of hosts
- Update or adding files to `playbooks/` for the lists of roles will be executed
- Execute compare
````
ansible-playbook playbooks/<file>.yml -i inventory/prod  --check --diff
````
- Apply

````
ansible-playbook playbooks/<file>.yml -i inventory/prod
````

2. Run Playbook [ Old ]

Update inventory host : ```inventory/prod/00-regional ```
```
[host]
workstation            ansible_host=115.79.25.9        ansible_port=6922     ansible_user=soulevil
vm01                   ansible_host=192.168.1.10        ansible_port=22     ansible_user=soulevil

```

```inventory/prod/01-ubuntu```
```
[host_ubuntu]
workstation

[vm]
vm01

[toanpt_test]
toanpr


```

Update playbook(list component deploy to server): ```playbooks/host_ubuntu.yml ```

```
- hosts: host_ubuntu
  become: yes
  roles:
  - { role: common, tags: common }
  - { role: node-exporter, tags: node-exporter }
  - { role: redis-exporter, tags: redis-exporter }
  - { role: memcached-exporter, tags: memcached-exporter }
  - { role: redis, tags: redis }
  - { role: mongodb-exporter, tags: mongodb-exporter }
  - { role: postgres-exporter, tags: postgres-exporter }
```

Verify: 
```
ansible-playbook playbooks/host_ubuntu.yml -i inventory/prod  --check --diff
```

![](https://i.imgur.com/b8rpTJH.png)

Apply:

```
ansible-playbook playbooks/host_ubuntu.yml -i inventory/prod
```
![](https://i.imgur.com/NDiCfkA.png)


