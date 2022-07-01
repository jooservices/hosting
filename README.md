## Overview: 

#### Monitoring
  - [x] Grafana 
  - [x] Prometheus
  - [ ] Alert manager
     - [ ] Send Email
     - [ ] Send Telegram

#### Exporter
  - [x] Node-exporter 
  - [x] Redis-exporter
  - [x] Memcached-exporter
  - [x] Redis-exporter
  - [x] Mongodb-exporter
  - [x] Postgres-exporter


#### Server tuning
  - [x] Install base packages
  - [x] Limits - raise fd hard/soft limits
  - [x] Set hostname
  - [x] Allow only root to control cron
  - [x] Disable selinux
  - [x] Postgres-exporter
  

## How to use Ansible 

1. Install ansible and require package use pip(3):
```
pip3 install -r requirements.txt
```
![](https://i.imgur.com/BiPnIcB.png)

2. Run Playbook

Update inventory host : ```inventory/prod/00-regional ```
```
[host]
workstation            ansible_host=115.79.25.9        ansible_port=6922     ansible_user=soulevil
vm01                   ansible_host=192.168.1.10        ansible_port=22     ansible_user=soulevil
toanpr                 ansible_host=192.168.1.7        ansible_port=22     ansible_user=toanpr

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

## Add new metrics to pormetheus

* Edit config / update target on prometheus
```
vim /etc/prometheus/prometheus.yml
```

![](https://i.imgur.com/CzuMOFK.png)

* Reload new config

```
curl -s -XPOST localhost:9090/-/reload
```

