---
title: "私有云安装"
linkTitle: "私有云安装"
edition: ce
weight: 1
description: >
  使用 ocboot 部署工具以 All in One 的方式部署私有云版本。
---

## 前提

{{% alert title="注意" color="warning" %}}
本章内容是通过部署工具快速搭建 Cloudpods 服务，如果想在生产环境部署高可用集群请参考: [高可用安装](../../setup/ha-ce/) 。
{{% /alert %}}

## 环境准备

### 机器配置要求

- 操作系统: 根据 CPU 架构不同，支持的发行版也不一样，目前支持的发行版情况如下：
    - [CentOS 7.6~7.9 Minimal](http://isoredirect.centos.org/centos/7/isos): 支持 x86_64 和 arm64
    - [Debian 10/11](https://www.debian.org/distrib/): 支持 x86_64 和 arm64
    - [Ubuntu 22.04](https://releases.ubuntu.com/jammy/): 仅支持 x86_64
    - [银河麒麟V10 SP2](https://www.kylinos.cn/scheme/server/1.html): 支持 x86_64 和 arm64
    - [统信 UOS kongzi](https://www.chinauos.com/): 支持 x86_64 和 arm64
- 操作系统需要是干净的版本，因为部署工具会重头搭建指定版本的 kubernetes 集群，所以确保系统没有安装 kubernetes, docker 等容器管理工具，否则会出现冲突导致安装异常
- 最低配置要求: CPU 4核, 内存 8GiB, 存储 100GiB
- 虚拟机和服务使用的存储路径都在 **/opt** 目录下，所以理想环境下建议单独给 **/opt** 目录设置挂载点
    - 比如把 /dev/sdb1 单独分区做 ext4 然后通过 /etc/fstab 挂载到 /opt 目录

## 安装ansible和git

首先需要安装ansible和git，ansible版本要求最低2.9.27，其中2.11.12测试较多。

{{< tabs name="ocboot_install" >}}
{{% tab name="CentOS 7" %}}

```bash
# 本地安装 ansible 和 git
$ yum install -y epel-release git python3-pip
$ python3 -m pip install --upgrade pip setuptools wheel
# 注意：请保留下面命令里的引号
$ python3 -m pip install 'ansible<=9.0.0'
```
{{% /tab %}}

{{% tab name="Kylin V10" %}}
```bash
# 本地安装 ansible 和 git
$ yum install -y git python3-pip
$ python3 -m pip install --upgrade pip setuptools wheel
# 注意：请保留下面命令里的引号
$ python3 -m pip install 'ansible<=9.0.0'
```
{{% /tab %}}

{{% tab name="Debian 10/11" %}}

如果提示`locale`相关的报错，请先执行：

```bash
if ! grep -q '^en_US.UTF-8' /etc/locale.gen; then
    echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen
    locale-gen
    echo 'LANG="en_US.UTF-8"' >> /etc/default/locale
    source /etc/default/locale
fi
```

```bash
# 本地安装 ansible 和 git
$ apt install -y git python3-pip
$ python3 -m pip install --upgrade pip setuptools wheel
# 注意：请保留下面命令里的引号
$ python3 -m pip install 'ansible<=9.0.0'
```
备注：已知在`debian 11`环境，如果`/proc/cmdline`里找不到启动选项 `systemd.unified_cgroup_hierarchy=0`，ocboot会自动配置相关的`GRUB`选项，重建启动参数，并重启操作系统，以便 `k8s` 能够正常启动。

{{% /tab %}}

{{% tab name="其它操作系统" %}}
```bash
# 本地安装 ansible
$ python3 -m pip install --upgrade pip setuptools wheel
# 注意：请保留下面命令里的引号
$ python3 -m pip install 'ansible<=9.0.0'
```
{{% /tab %}}

{{< /tabs >}}

## 安装Cloudpods

部署的工具在 [https://github.com/yunionio/ocboot](https://github.com/yunionio/ocboot)，需要把该工具使用 `git clone` 下来，然后运行 `run.py` 脚本部署服务。操作步骤如下:

```bash
# 下载 ocboot 工具到本地
$ git clone -b {{<release_branch>}} https://github.com/yunionio/ocboot && cd ./ocboot
```

执行 run.py 部署服务。其中 **<host_ip>** 为部署节点的 IP 地址，该参数为可选项。如果不指定则选择默认路由出去的那张网卡部署服务。如果你的节点有多张网卡，可以通过指定 **<host_ip>** 选择对应网卡监听服务。

{{< tabs name="ocboot_install_region" >}}
{{% tab name="中国大陆" %}}

```bash
# 直接部署，会从 registry.cn-beijing.aliyuncs.com 拉取容器镜像
$ ./run.py virt <host_ip> 

# 如果遇到 pip 安装包下载过慢的问题，可以用 -m 参数指定 pip 源
# 比如下面使用: https://mirrors.aliyun.com/pypi/simple/ 源
$ ./run.py -m https://mirrors.aliyun.com/pypi/simple/ virt <host_ip> 
```

{{% /tab %}}

{{% tab name="其他地区" %}}

对于某些网络环境，registry.cn-beijing.aliyuncs.com 访问缓慢或不可达，在版本 `v3.9.5`之后（含），可指定镜像源：[docker.io](http://docker.io) 来安装。命令如下：

```bash
IMAGE_REPOSITORY=docker.io/yunion ./run.py virt <host_ip>
```

{{% /tab %}}

{{< /tabs >}}

## 部署完成

```bash
....
# 部署完成后会有如下输出，表示运行成功
# 浏览器打开 https://10.168.26.216 ，该 ip 为之前设置 <host_ip>
# 使用 admin/admin@123 用户密码登录就能访问前端界面
Initialized successfully!
Web page: https://10.168.26.216
User: admin
Password: admin@123
```

然后用浏览器访问 https://10.168.26.216 ，用户名输入 `admin`，密码输入 `admin@123` 就会进入 Cloudpods 的界面。

![登录页](../images/index.png)

## 开始使用Cloudpods

### 创建第一台虚拟机

通过如下三步创建出第一台虚拟机：

#### 1. 导入镜像

浏览位于 [CentOS 7云主机镜像](https://cloud.centos.org/centos/7/images/) ，选择一个GenericCloud 镜像，拷贝镜像URL。

在 `主机` 菜单，选择 `系统镜像`，选择 `上传`。输入镜像名称，选择 `输入镜像URL`，粘贴上述CentOS 7镜像URL，选择 `确定`。

可以访问 https://docs.openstack.org/image-guide/obtain-images.html 获得更多的虚拟机镜像。

#### 2. 创建网络（VPC和IP子网）

- 新建VPC]: 在 `网络` 菜单，选择 `VPC` 子菜单，选择 `新建`。输入名称，例如 `vpc0`，选择目标网段，例如 `192.168.0.0/16`。点击 `新建`。

- 新建IP子网: VPC创建完成后，选择 `IP子网` 子菜单，选择 `新建`。输入名称，例如 `vnet0`，选择VPC为刚才创建的VPC `vpc0`，选择可用区，输入 `子网网段`，例如 `192.168.100.0/24`。点击 `新建`。

[典型网络配置](../../function_principle/onpremise/network/examples)提供了几种常见的宿主机网络配置，请参考。

#### 3. 创建虚拟机

在 `主机` 菜单，选择 `虚拟机`，选择 `新建`。在此界面输入主机名，选择镜像和IP子网，创建虚拟机。

## FAQ

### 1. 在 All in One 部署完成后宿主机列表没有宿主机？

如下图所示，若发现环境部署完成后宿主机列表中没有宿主机，可按照以下方式进行排查

  ![](../images/nohost.png)


1. 在控制节点排查 host 问题，请参考：[Host服务问题排障技巧](../../function_principle/onpremise/host/troubleshooting/)


    1. 若日志报错信息中包含“register failed: try create network: find_matched == false”，则表示未成功创建包含宿主机的IP子网，导致宿主机注册失败，请创建包含宿主机网段的IP子网。

    ```
    # 创建包含宿主机网段的IP子网
    $ climc network-create bcast0  adm0 <start_ip> <end_ip> mask
    ```

    ![](../images/iperror.png)

    2. 若日志报错信息中包含“name starts with letter, and contains letter, number and - only”，则表示宿主机的主机名不合规，应改成以字母开头的hostname

    ![](../images/hostnameerror.png)

### 2. 在 All in One 中找不到虚拟机界面？

All in One 部署的节点会部署 Cloudpods host 计算服务，作为宿主机，具有创建和管理私有云虚拟机的能力。没有虚拟机界面应该是 Cloudpods 环境中没有启用宿主机。

请到 `管理后台` 界面，点击 `主机/基础资源/宿主机` 查看宿主机列表，启用相应的宿主机，刷新界面就会出现虚拟机界面。

{{% alert title="注意" color="warning" %}}
如果要使用 Cloudpods 私有云虚拟机，并且宿主机是 CentOS 7 的发行版。需要宿主机使用 Cloudpods 编译的内核，可使用以下命令查看宿主机是否使用 Cloudpods 内核(包含 yn 关键字)。

```bash
# 查看是否使用 yn 内核
$ uname -a | grep yn
Linux office-controller 3.10.0-1160.6.1.el7.yn20201125.x86_64
# 如果内核不是带有 yn 关键字的版本，可能是第一次使用 ocboot 安装，重启即可进入 yn 内核
$ reboot
```
{{% /alert %}}

![宿主机](../images/host.png)

### 3. 为什么修改了节点的 hostname ，服务启动不了了？

Cloudpods 底层使用了 Kubernetes 管理节点，Kubernetes 节点名称依赖 hostname，改了 hostname 会导致节点无法注册到 Kubernetes 集群，所以不要修改 hostname ，如果修改了，请改之前的名称，服务就会自动恢复了。

### 4. 如何重装?

1. 执行 `kubeadm reset -f` 删除 kubernetes 集群

2. 重新运行 ocboot 的 run.py 脚本

### 5. 其它问题？

其它问题欢迎在 Cloudpods github issues 界面提交: [https://github.com/yunionio/cloudpods/issues](https://github.com/yunionio/cloudpods/issues) , 我们会尽快回复。
