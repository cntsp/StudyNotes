# 搭建K8S集群

## 搭建k8s环境平台规划

### 单master集群

单个master节点，然后管理多个node节点

![](images/image-20200928110456495-1612246108205.png)

### 多master集群

多个master节点，管理多个node节点

![](../../../../%E7%BB%83%E4%B9%A0%E6%96%87%E4%BB%B6%E5%A4%B9/docker/images/image-20200928110543829-1612246131570.png)

## 服务器硬件配置要求

### 测试环境

master：2核 4G 20G

node：4核 8G 40G



### 生产环境

master：8核 16G 100G

node：16核 64G 200G



目前生产部署Kubernetes 集群主要有两种方式

### kubeadm

kubeadm 是一个K8S部署工具，提供kubeadm init 和 kubeadm join, 用于快速部署kubernetes集群

官网地址：[点我传送](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### 二进制包

从github下载发行版的二进制包，手动部署每一个组件，组成kubernetes集群。

kubeadm部署门槛低，但是屏蔽了各个组件之间的交互细节，遇到问题很难定位和排查，如果想更容易可控，推荐使用二进制包部署kubernetes集群，虽然手动部署麻烦点，但是这个过程期间你可以更好的了解每个组件的作用，也利用后期维护。不过kubeadm可是kubernetes官方推荐的方式，随着版本的迭代，相信kubeadm方式在生成环境的使用也会越来越普遍。

## kubeadm部署集群

kubeadm 是官方社区推出一个用于快速部署kubernetes 集群的工具，这个工具能通过两条指令完成一个kubernetes 集群的部署:

- 创建一个Mater 节点 kubeadm init
- 将Node节点加入到当前集群中 $ kubeadm join <Mater 节点的IP和端口>



## 安装要求

在开始之前，部署kubernetes集群机器需要满足以下几个条件

- 一台或多台机器，操作系统为Centos7.X
- 硬件配置：2GB或更多GAM
- 集群中所有机器之间网络互通
- 可以访问外网，需要拉取镜像
- 禁止swap分区







































