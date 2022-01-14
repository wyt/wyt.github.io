---
title: Powershell会话中设置环境变量
date: 2018-10-30 10:30:00
updated: 2018-10-30 10:30:00
comments: true
---

### Powershell会话中设置环境变量
``` bash
PS C:\personal\idea\cloud-cook> ls env:DOCKER_HOST

Name                           Value
----                           -----
DOCKER_HOST                    tcp://192.168.1.114:2375


PS C:\personal\idea\cloud-cook> $env:DOCKER_HOST="tcp://192.168.21.129:2375"   # 在当前会话中创建DOCKER_HOST环境变量
PS C:\personal\idea\cloud-cook> ls env:DOCKER_HOST

Name                           Value
----                           -----
DOCKER_HOST                    tcp://192.168.21.129:2375
```

More info: [Powershell环境变量](https://www.pstips.net/powershell-environment-variables.html)