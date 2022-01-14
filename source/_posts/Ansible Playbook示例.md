---
title: Ansible Playbook安装docker
date: 2019-7-11 13:32:18
categories:
- ansible
tags:
- ansible
- playbook
---

展示一个安装docker示例，Ansible playbook内容如下：

``` javascript
---
- hosts: all
  tasks:
  - name: Remove docker
    yum:
      name: ['docker', 'docker-client', 'docker-client-latest', 'docker-common', 'docker-latest', 'docker-latest-logrotate', 'docker-logrotate', 'docker-selinux', 'docker-engine-selinux']
      state: removed
  - name: Install yum utils
    yum:
      name: ['yum-utils', 'device-mapper-persistent-data', 'lvm2']
      state: installed
  - name: Set aliyun repo
    shell: yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo;yum makecache fast
  - name: Install docker-ce
    yum: name=docker-ce state=installed
  - name: Registry mirrors
    script: ./registry_mirrors.sh
  - name: After registry mirrors
    shell: systemctl daemon-reload;systemctl restart docker;systemctl enable docker
  - name: Show docker version
    command: docker -v
    register: result
  - name: Debug info
    debug: msg='{{result.stdout_lines}}'
```

registry_mirrors.sh脚本内容

``` bash
#!/usr/bin/env bash

mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://kc0hk0ee.mirror.aliyuncs.com"]
}
EOF
```

执行

``` bash
ansible-playbook install_docker.yml
```


More info: [Docker install](https://yongtao.wang/2018/10/25/Docker%20install/)
