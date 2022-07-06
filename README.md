## Overview: 

#### Monitoring
  - [x] Grafana 
  - [x] Prometheus
  - [x] Alert manager
     - [x] Send Slack
     - [x] Send Telegram

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

## App usage
- Grafana : Display graph for monitoring
- Prometheus : Collect data from exporter and queryable by another services
- Alert : Sending alert
- Exporter : Read data from `sources` and export for 3rd party

![image](https://user-images.githubusercontent.com/2688707/177316892-e3d3b862-6f82-4fa9-a56d-e53a4502ef08.png)

## Prometheus
https://github.com/prometheus/prometheus/

https://prometheus.io/docs/prometheus/latest/getting_started/

- How to install
https://computingforgeeks.com/install-prometheus-server-on-debian-ubuntu-linux/

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



## How to add alert rule / config routing.
1. All alert rule store at ```/etc/prometheus/rules```

```
root@workstation:/home/soulevil# ls /etc/prometheus/rules
host.yaml  mongodb.yaml  mysql.yaml  postgresql.yaml  redis.yaml
```

Example host.yaml

```
groups:
- name: host
  rules:
  

  - alert: HostOutOfMemory
    expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 > 10
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Host out of memory (instance {{ $labels.instance }})
      description: "Node memory is filling up (< 10% left)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

  - alert: HostMemoryUnderMemoryPressure
    expr: rate(node_vmstat_pgmajfault[1m]) > 1000
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Host memory under memory pressure (instance {{ $labels.instance }})
      description: "The node is under heavy memory pressure. High rate of major page faults\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

  - alert: HostUnusualNetworkThroughputIn
    expr: sum by (instance) (rate(node_network_receive_bytes_total[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Host unusual network throughput in (instance {{ $labels.instance }})
      description: "Host network interfaces are probably receiving too much data (> 100 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"


  - alert: HostUnusualNetworkThroughputOut
    expr: sum by (instance) (rate(node_network_transmit_bytes_total[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Host unusual network throughput out (instance {{ $labels.instance }})
      description: "Host network interfaces are probably sending too much data (> 100 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"


  - alert: HostUnusualDiskReadRate
    expr: sum by (instance) (rate(node_disk_read_bytes_total[2m])) / 1024 / 1024 > 50
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Host unusual disk read rate (instance {{ $labels.instance }})
      description: "Disk is probably reading too much data (> 50 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"


  - alert: HostUnusualDiskWriteRate
    expr: sum by (instance) (rate(node_disk_written_bytes_total[2m])) / 1024 / 1024 > 50
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Host unusual disk write rate (instance {{ $labels.instance }})
      description: "Disk is probably writing too much data (> 50 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"


  - alert: HostHighCpuLoad
    expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 80
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: Host high CPU load (instance {{ $labels.instance }})
      description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"


  - alert: HostPhysicalComponentTooHot
    expr: node_hwmon_temp_celsius > 75
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Host physical component too hot (instance {{ $labels.instance }})
      description: "Physical hardware component too hot\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

```

2. Reload new config:

After add new or modify alert rule, we need reload it:

```
curl -s -XPOST localhost:9090/-/reload
```


3. Check alert rules on Prometheus UI

p/s: add hosts ```115.79.25.9 prometheus.xcrawler.net alertmanager.xcrawler.net```

Basic Authen - Find it add haproxy config: ```userlist AuthUsers```

**- Alert rules:**

Link: http://prometheus.xcrawler.net/alerts

![](https://i.imgur.com/1hi88tW.png)


**- Alert routing:**

Link: http://alertmanager.xcrawler.net/#/alerts
![](https://i.imgur.com/kIF8ik2.png)


4. Ref:

Collection of alerting rules: https://awesome-prometheus-alerts.grep.to/rules.html

#### Nginx config
> With SSL-Pass thru, Nginx is dealing in encrypted TCP traffic - it does not decrypt it, and cannot read information about the HTTP request. It's job is merely to send TCP packets to other servers based on it's load balancing configuration. This has some side affects - notably that Nginx can't figure out what server to send traffic to based on the Host header (although SNI can get around that - that's a topic for another day).
1. Add new website(Example will need add new website with name ```test.xcrawler.net```)
* Edit proxy config at: ```/etc/nginx/proxy.conf```
* Add new map and upstream
```
    map $ssl_preread_server_name $targetBackend {
        develop.coffeeschool.vn   shared;
        develop.baristaschool.vn  shared;
        phongcachbarista.vn       shared;
        xcrawler.net  xcrawler;
        test.xcrawler.net  testxcrawler;
    }
```
```
upstream testxcrawler {
        server 192.168.1.4:443 weight=5;

        ## Add more if loadbalancing needed
        server 192.168.1.5:443 max_fails=3 fail_timeout=30s;
        server 192.168.1.6:443 ;
    }
```

![](https://i.imgur.com/8onPJ1U.png)

2. Verify config and reload.
```
root@workstation:/home/soulevil# /usr/sbin/nginx -t -c /etc/nginx/nginx.conf
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
root@workstation:/home/soulevil# systemctl reload nginx
```

3. REF
