# ansible-role-mysql-lb

Ansible role files for mysql load balancers

Tested on Ubuntu 14.04

## Role variables

Default variables are in `defaults/main.yml`.

This role has just one default variable now.
```
  mysql_check_user: haproxy_check
```

### mysql_instances

List of mysql instances. All members should have three keys, name, host, and port.
```
  mysql_instances:
    - { name: mysql1, host: 192.168.100.51, port: 3306 }
    - { name: mysql2, host: 192.168.100.50, port: 3306 }
```

### virtual_ip

Virtual IP which master keepalived binds to.
```
  virtual_ip: 192.168.100.60
```

### keepalived

Each loadbalancer should have different keepalived configuration for failover.

This variable has following keys.

`role`: role of keepalived. MASTER or BACKUP

`interface`: network interface

`virtual_router_id`: id of keepalived for vrrp

`priority`: priority value used for master election.

Sample variable of master instance is below.
```
  keepalived:
    role: MASTER
    interface: eth0
    virtual_router_id: 100
    priority: 200
```
And backup instance is
```
  keepalived:
    role: BACKUP
    interface: eth0
    virtual_router_id: 101
    priority: 100
```

## Example playbooks

### Example1

All variables written in a playbook.
```
  - name: Install haproxy and configure for load balancers
    hosts: first-lb.example.com
    become: yes
    roles:
      - role: mysql-lb
        mysql_instances:
          - { name: mysql1, host: 192.168.100.51, port: 3306 }
          - { name: mysql2, host: 192.168.100.50, port: 3306 }
        virtual_ip: 192.168.100.60
        keepalived:
          role: MASTER
          interface: eth0
          virtual_router_id: 100
          priority: 200
```

### Example2

Use group and host variables if you want to configure two or more load balancers.

Assume mysql-lb-group consists of two hosts, mysql-lb-a and mysql-lb-b.

sample_playbook.yml
```
  - name: Install haproxy and configure for load balancers
    hosts: mysql-lb-group
    become: yes
    roles:
      - mysql-lb
```

group_vars/mysql-lb-group
```
  mysql_instances:
    - { name: mysql1, host: 192.168.100.51, port: 3306 }
    - { name: mysql2, host: 192.168.100.50, port: 3306 }
  virtual_ip: 192.168.100.60
```

host_vars/mysql-lb-a
```
  keepalived:
    role: MASTER
    interface: eth0
    virtual_router_id: 100
    priority: 200
```
host_vars/mysql-lb-b
```
  keepalived:
    role: BACKUP
    interface: eth0
    virtual_router_id: 101
    priority: 100
```
