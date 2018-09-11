# RadonDB Ansible 部署

## 概述

Ansible是一个简单的自动化运维管理工具，基于Python语言实现。radon-ansible是基于Ansible Playbook功能编写的集群自动化部署工具，可以快速部署一套完整的Radon集群。

目前已实现功能如下：

- 部署集群
- 启动集群
- 初始化集群
- 关闭集群

## 环境要求

  - ubuntu Server 16.04 LTS 64bit
  - radondb推荐配置：1个radon SQL节点，2个radon存储节点，1个radon计算节点,1台监控机器
  - xenon和keeplived部署有待开发，因此各节点暂不支持部署主从。
  - 准备一台中控机
  - 机器之间内网互通，最好可以连接外网

## 准备工作

1. 安装依赖包

执行以下命令：

```
# sudo apt-get -y install git curl sshpass python-pip
```

2. 创建用户
