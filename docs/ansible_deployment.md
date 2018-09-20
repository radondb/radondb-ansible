目录
=================

* [RadonDB Ansible 部署](#radonDB-ansible-部署)
   * [概述](#概述)
   * [环境要求](#环境要求)
   * [准备工作](#准备工作)   
      * [安装依赖包](#安装依赖包)
      * [创建用户](#创建用户)
      * [生成 ssh key](#生成-ssh-key)
      * [下载部署文件](#下载部署文件)
      * [安装Ansible](#安装ansible)
      * [修改配置文件](#修改配置文件)
   * [下载安装包](#下载安装包)
   * [部署集群](#部署集群)
   * [启动集群](#启动集群)
   * [初始化集群](#初始化集群)
   * [测试集群](#测试集群)
   * [关闭集群](#关闭集群)

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
  - 一台可连接外网的中控机
  - 机器之间内网互通

## 准备工作

### 安装依赖包

在中控机上执行以下命令：

```
# sudo apt-get -y install git curl sshpass python-pip
```

### 创建用户

各主机创建一个`ubuntu`用户，并设置密码。用户名由`inventory.ini`中的参数`ansible_user`指定，部署将在该用户下运行。

```
# useradd -m -d /home/ubuntu ubuntu

# passwd ubuntu
```

### 生成 ssh key

在`ubuntu`用户下，执行以下指令生成ssh密钥，遇到提示直接回车即可。

```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:EDraeJ3YoP5Bv/Wv1z6Mvj5ukuJRYOh/Jyv/ObwhHJ0 ubuntu@mysql
The key's randomart image is:
+---[RSA 2048]----+
|      .          |
|     . o         |
|    + o o        |
|   = B + . . .   |
|  + = = S o E    |
| . o . . o .     |
|  . . . + =oo+   |
|   . . oo+o=Bo+  |
|    . ...+=@X*.. |
+----[SHA256]-----+
```

### 下载部署文件

- 在`ubuntu`用户下，从`github`下载`radondb-ansible`。

```
# git clone https://github.com/radondb/radondb-ansible.git
```

- 或者，从`github`下载`radondb-ansible`的release版。

```
# wget https://github.com/radondb/radondb-ansible/releases/download/0.1.0/radondb-ansible-0.1.0.tar.gz
# tar -xvf radondb-ansible-0.1.0.tar.gz
```

### 安装Ansible

在`radondb-ansible`目录下执行以下指令安装Ansible及其依赖程序。

```
$ cd /home/ubuntu/radondb-ansible
$ sudo pip install -r ./requirements.txt
```

### 修改配置文件

`radondb-ansible`目录下的`inventory.ini`文件是各种部署配置信息。规划好集群部署计划，配置主机列表和变量。

主机清单：

| 主机名  | 含义   |   其他              |
| -------------- | ----------------- |--------- |
| radon:master | radon主SQL节点 |  必须配置 |
| radon:slave | radon备SQL节点 | 主备需手动配置 |
| mysql:backend | radondb存储节点 | 至少一个，配置server_id |
| mysql:backup | radondb计算节点 | 配置server_id |
| monitor | 监控 | 一个节点 |

例如：计划192.168.0.25和192.168.0.26作为radondb的存储节点，192.168.0.27作为radondb的计算节点，192.168.0.28作为radondb的SQL节点，192.168.0.29作为监控节点，则其主机清单配置如下：

```
[radon:children]
master
slave
[master]
192.168.0.28
[slave]

[mysql:children]
backend
backup
###server_id must be defferent
[backend]
192.168.0.25 server_id=19216825
192.168.0.26 server_id=19216826
[backup]
192.168.0.27 server_id=19216827
[monitor]
192.168.0.29
```

radon主机组配置参数：

| 变量 | 含义 |
| ---- | ------ |
| deploy_dir | radon部署目录 |
| deploy_usr | radon部署用户 |
| endpoint | radon监听端口 |
| twopc_enable | 是否启用分布式事务 |
| max_connections | 最大连接数 |
| max_result_size | 单个存储节点上最大结果集的大小（单位：字节），0表示无限制。 |
| ddl_timeout | DDL超时时间，0表示无限制(ms) |
| query_timeout | 查询超时时间，0表示无限制(ms) |
| mode | 审计模式;None: 不开启审计；Audit-Read：审计读；Audit-Write：审计写；Audit-Read-Write：审计读和写 |
| monitor_port | 监控metrics监听端口 |
| radon_usr | RadonDB用户名 |
| radon_passwd | RadonDB密码 |

mysql主机组配置参数：

| 变量 | 含义 |
| ---- | ------ |
| deploy_dir | mysql部署目录 |
| deploy_usr | mysql部署用户 |
| mysql_data_dir | mysql data目录 |
| max_connections | 最大连接数 |
| mysqld_exporter_usr | mysqld_exporter监控使用用户名 |
| mysqld_exporter_passwd | mysqld_exporter监控使用密码 |
| mysqld_exporter_port | mysqld_exporter监控端口 |

monitor主机组配置参数：

| 变量 | 含义 |
| ---- | ------ |
| prometheus_dir | prometheus部署目录 |
| prometheus_port | prometheus端口 |
| grafana_dir | grafana 部署目录 |
| grafana_port | grafana 监听端口 |
| grafana_admin_user | grafana 用户名 |
| grafana_admin_password | grafana 密码 |

Global 配置参数：

| 变量 | 含义 |
| ---- | ------ |
| ansible_python_interpreter | 指定python 解释器路径 |
| ansible_user | 部署执行用户 |
| group_name | 部署用户所属组 |
| exporter_dir | exporter部署目录 |
| cluster_name | 集群名称 |
| retry_stagger | 重试次数 |
| mysql_port | mysql端口 |
| mysql_user | mysql用户名 |
| mysql_passwd | mysql密码 |
| node_exporter_port | node_exporter端口 |

## 下载安装包

在radondb-ansible 目录下执行：

```
# ansible-playbook -i inventory.ini local_prepare.yml -K
```

输入sudo密码后开始下载部署需要的`radon`、`mysql`、`prometheus`、`grafana`、`node_exporter`、`mysqld_exporter`等安装包。目录如下：

```
resources/
├── grafana-5.2.3.tar.gz
├── libaio1-0.3.110.deb
├── libaio-dev-0.3.110.deb
├── limits.conf
├── mysqld_exporter
├── node_exporter
├── Percona-Server-5.7.22-22.tar.gz
├── prometheus
├── radon
├── radoncli
└── sysctl.conf
```

## 部署集群

在radondb-ansible 目录下执行：

```
# ansible-playbook -i inventory.ini deploy.yml -k -K
```

输入ssh和sudo密码后开始执行。

## 启动集群

在radondb-ansible 目录下执行：

```
# ansible-playbook -i inventory.ini start.yml -K
```

输入sudo密码后开始执行。

## 初始化集群

初次部署并启动radondb集群后,需要执行：

```
# ansible-playbook -i inventory.ini init.yml -K
```

初始化包括配置radon集群，配置grafana的数据源，上传dashboards等。此步骤只在初次部署时执行。

## 测试集群

-   使用 MySQL 客户端连接测试，3306为radon默认端口，默认用户为usr。

    ```
    mysql -u usr -h 192.168.0.28 -P 3306 -p
    ```

-   通过浏览器访问监控平台。

    地址：`http://192.168.0.29:3000`  默认帐号密码是：`admin`/`admin`

## 关闭集群

```
# ansible-playbook -i inventory.ini stop.yml -K
```
