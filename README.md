# 开源BI工具安装配置文档

本文档包含了[JupyterLab](#JupyterLab配置)、[SuperSet](#JupyterLab配置)、[Zeppelin](#SuperSet的配置)在CENTOS 7的安装配置

## OS环境配置

- 安装必要的系统组件

``yum install -y nano net-tools wget bzip2 gcc gcc-c++ python-devel cyrus-sasl-devel cyrus-sasl-plain  cyrus-sasl-gssapi krb5-workstation krb5-libs krb5-auth-dialog ntp ntpdate``

- 配置时钟同步

``ntpdate cn.pool.ntp.org``

``hwclock --systohc``

- 关闭防火墙服务

``systemctl stop firewalld.service``

``systemctl disable firewalld.service``

- krb客户端配置

 修改krb5.conf

``nano /etc/krb5.conf``

将以下配置文件内容粘贴到``/etc/krb5.conf``

```
 # Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
# pkinit_anchors = /etc/pki/tls/certs/ca-bundle.crt
 default_realm = SGMW.COM
# default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 SGMW.COM = {
  kdc = cm.sgmw.com
  admin_server = cm.sgmw.com
 }

[domain_realm]
 .sgmw.com = SGMW.COM
sgmw.com = SGMW.COM

```

  修改hosts

``nano /etc/hosts``

将以下配置文件内容粘贴到``nano /etc/hosts``

``10.1.126.229    cm.sgmw.com``

``10.1.126.235    cnwulcdhnode01``

  测试krb配置
  
``kinit finance/remote@SGMW.COM``
 
``klist``

## Miniconda配置
- 下载Miniconda3安装程序

``wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh``

- 运行Miniconda3安装程序

``bash Miniconda3-latest-Linux-x86_64.sh``

按回车继续，遇到问题回答yes，执行下面命令让配置生效：

``source /root/.bashrc``

- 添加conda国内镜像：

``conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/``

``conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/``

``conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/``

``conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/menpo/``

``conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/``

- 添加pip国内镜像

```
[root@localhost ~]# cd ~
[root@localhost ~]# mkdir .pip
[root@localhost ~]# cd .pip
[root@localhost .pip]# nano pip.conf
```

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

- 更新conda版本

``conda update -n base -c defaults conda``

## JupyterLab配置

- 使用 [jupyterlab.yaml ](https://github.com/chenxixian/BiToolsSetup/raw/master/jupyterlab.yaml)建立jupyterlab虚拟环境

``conda env create -f jupyterlab.yaml``

- 进入jupyterlab虚拟环境

``conda activate jupyterlab``


- 运行jupyterlab

``LANG=zn jupyter lab --port 8888 --no-browser --allow-root --ip='192.168.2.220'``


## SuperSet的配置

- 使用 [superset.yaml](https://github.com/chenxixian/BiToolsSetup/raw/master/superset.yaml)建立superset虚拟环境

``conda env create -f superset.yaml``

- 进入superset虚拟环境

``conda activate superset``


- 配置superset

``fabmanager create-admin --app superset``

``superset db upgrade``

``superset load_examples``

``superset init``

- 运行superset

``superset runserver -p 8388``

## Zeppelin的配置

``yum list installed | grep java``

- 安装JDK

``yum -y install java-1.8.0-openjdk*``

- 安装Zeppelin

``tar -zxvf zeppelin-0.8.1-bin-all.tgz``

- 运行Zeppelin

``cd zeppelin-0.8.1-bin-all``

``bin/zeppelin-daemon.sh start``
