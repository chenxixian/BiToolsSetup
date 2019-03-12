# 常用BI工具安装配置文档

本文档包含了[JupyterLab](#JupyterLab配置)、[SuperSet](#JupyterLab配置)、[Zeppelin](#SuperSet的配置)、[FineBi](#FineBi的配置)、[Tableau Server](#Tableau_Server的配置) 在CENTOS 7的安装配置

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

- 访问http://ip:8888 ,以Python为Kernel，新建一个Notebook，粘贴代码执行：

```
from impala.dbapi import connect
conn = connect(host='cnwulcdhnode01', port=21050,database='dm_finance',auth_mechanism='GSSAPI',kerberos_service_name='impala')
cursor = conn.cursor()
cursor.execute('select company_number,sum(price) from dm_finance.dw_col_taxcheck group by company_number ;')
#print (cursor.description) # prints the result set's schema
#results = cursor.fetchall()
#print (results)
from impala.util import as_pandas
df = as_pandas(cursor)
df.head(3)
```

- 编写一个启动脚本：startjupyterlab.sh

```
cat ~/.bashrc > startjupyterlab.sh
echo "conda activate jupyterlab && LANG=zn jupyter lab --port 8888 --no-browser --allow-root --ip=10.3.58.19" >> startjupyterlab.sh
echo "conda deactivate" >> startjupyterlab.sh
```
运行启动脚本： ./startsuperset.sh

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

- 配置用impala连接数据库

访问http://ip:8388 , 修改与原为中文，点“数据源”，“数据库”，“+”：

数据库

``impala_dm_finance``

SQLAlchemy URI

``impala://10.1.126.235:21050/dm_finance``

在 SQL 工具箱中公开，选上

扩展

```
{
    "metadata_params": {},
    "engine_params": {
      "connect_args": {
        "host": "cnwulcdhnode01",
        "auth_mechanism":"GSSAPI",
         "kerberos_service_name": "impala"
        }
},
    "metadata_cache_timeout": {},
    "schemas_allowed_for_csv_upload": []
}

```

点测试连接来测试，成功后保存

- 编写一个启动脚本：startsuperset.sh

```
cat ~/.bashrc > startsuperset.sh
echo "conda activate superset && superset runserver -p 8388" >> startsuperset.sh
echo "conda deactivate" >> startsuperset.sh
```
运行启动脚本： ./startsuperset.sh

## Zeppelin的配置

``yum list installed | grep java``

- 安装JDK

``yum -y install java-1.8.0-openjdk*``

- 安装Zeppelin

``tar -zxvf zeppelin-0.8.1-bin-all.tgz``

- 运行Zeppelin

``cd zeppelin-0.8.1-bin-all``

``bin/zeppelin-daemon.sh start``

- 配置Interpreter

访问ip:8080,进入Interpreter界面 ，点击 + 创建一个interpreter：

```
Interpreter Name: impala
Interpreter group: jdbc
```


需要把finance-remote.keytab放在/root/

修改配置参数：

```
default.driver com.cloudera.impala.jdbc41.Driver
default.url jdbc:impala://10.1.126.235:21050;AuthMech=1;KrbHostFQDN=cnwulcdhnode01;KrbServiceName=impala;
zeppelin.jdbc.auth.type KERBEROS
zeppelin.jdbc.keytab.location /root/finance-remote.keytab
zeppelin.jdbc.principal finance/remote@SGMW.COM
zeppelin.jdbc.interpolation true
```

新增配置参数：

```
zeppelin.jdbc.auth.kerberos.proxy.enable true
default.proxy.user.property DelegationUID
```

将JDBC的驱动（ [ImpalaJDBC41.jar](https://github.com/chenxixian/BiToolsSetup/raw/master/ImpalaJDBC41.jar) ）复制到Interpreter的位置：

``/root/zeppelin-0.8.1-bin-all/interpreter/jdbc``

把Hadoop安装源中的依赖文件,复制到Interpreter的位置：

``cp /opt/cloudera/parcels/CDH/lib/hadoop/client/*.jar /root/zeppelin-0.8.1-bin-all/interpreter/jdbc/``

- 重启zeppelin

``bin/zeppelin-daemon.sh stop``
``bin/zeppelin-daemon.sh start``

- 访问ip:8080,新建一个Notebook，粘贴代码执行：

```
%impala
select company_number,sum(price) from dm_finance.dw_col_taxcheck group by company_number 
```

- 编写一个启动脚本：startzeppelin.sh

```
echo "cd zeppelin-0.8.1-bin-all" > startzeppelin.sh
echo "bin/zeppelin-daemon.sh start" >> startzeppelin.sh
echo "tail -f logs/zeppelin-root-localhost.localdomain.log" >> startzeppelin.sh
```
运行启动脚本： ./startzeppelin.sh

## FineBi的配置

- 下载FineBi

`` wget http://fine-build.oss-cn-shanghai.aliyuncs.com/finebi/5.1/stable/exe/spider/linux_unix_FineBI5_1-CN.sh``

- 安装FineBi

``bash linux_unix_FineBI5_1-CN.sh``

按实际情况回答

- 运行FineBi

开一个控制台运行：

```
tail -f /usr/local/FineBI5.1/logs/fanruan.log
```

开另一个控制台运行：

```
bash /usr/local/FineBI5.1/bin/finebi
```

访问：http://ip:37799/webroot/decision/

- 配置KERBEROS impala连接

将JDBC的驱动（ [ImpalaJDBC41.jar](https://github.com/chenxixian/BiToolsSetup/raw/master/ImpalaJDBC41.jar) ）复制到lib的位置：

``/usr/local/FineBI5.1/webapps/webroot/WEB-INF/lib``

需要把finance-remote.keytab放在/root/,执行：

``kinit -kt /root/finance-remote.keytab finance/remote@SGMW.COM``

在FineBi后台页面配置impala连接：

````
URL
jdbc:impala://10.1.126.235:21050/dm_finance;AuthMech=1;KrbHostFQDN=cnwulcdhnode01;KrbServiceName=impala;LogLevel=6;LogPath=/root/logs;

认证方式
kerberos

客户端principal
finance/remote@SGMW.COM

keytab密钥路径
/root/finance-remote.keytab 

````

# Tableau_Server的配置

- 系统需求

官方建议 
CPU 不低于 8 核 
内存最少 8G （低于 8G 无法安装） 
硬盘空间建议不低于 50G 

- 建立应用管理员用户

必须用非 root 用户安装

建立新用户 tabadmin 并分配给用户组 tsmadmin

```
useradd tabadmin
passwd tabadmin
groupadd tsmadmin

usermod -G tsmadmin -a tabadmin
```

给与 tabadmin 账户 sudo 权限

``nano /etc/sudoers``

添加以下内容

``tabadmin ALL=(ALL) ALL``

切换到 tabadmin 账户

``su tabadmin``

- 下载安装包并安装
```
sudo wget https://www.tableau.com/downloads/server/rpm
sudo yum install tableau-server-2019-1-1.x86_64.rpm
cd /opt/tableau/tableau_server/packages/scripts.20191.19.0215.0259/
sudo ./initialize-tsm --accepteula
```
安装完成后切换成 root 账户，或其他带 sudo 权限的账户

用 tsm 命令登陆

``tsm login -u tabadmin``

激活 Key(如果有Key)

``tsm licenses activate -k``

或者申请试用 Trail

``tsm licenses activate -t``

- 注册

创建注册配置文件

````
tsm register --template > /root/registration_file.json

nano /root/registration_file.json

{
"zip" : "03079",
"country" : "USA",
"city" : "Salem",
"last_name" : "Smith",
"industry" : "Software",
"eula" : "yes",
"title" : "Software Applications Engineer",
"phone" : "5556875309",
"company" : "Example",
"state" : "NH",
"department" : "Engineering",
"first_name" : "Jason",
"email" : "jsmith@example.com"
}
````

注册文件传递给 TSM 以注册

``tsm register --file /root/registration_file.json``

身份验证（这次用本地身份验证）

```
nano /root/local_auth_file.json

{
"configEntities":{
"identityStore": {
"_type": "identityStoreType",
"type": "local"
}
}
}
```

传递配置文件

``tsm settings import -f /root/local_auth_file.json``

应用更改

``tsm pending-changes apply``

- 初始化 Tableau Server 。

若要初始化并启动 Tableau Server ，请使用 –start-server 选项：

``tsm initialize --start-server --request-timeout 1800``

这样将能在初始化后保持服务器运行，从而节省时间。

如果您打算在初始化后重新配置 Tableau Server ，请关闭 –start-server 选项：

``tsm initialize --request-timeout 1800``
这将在初始化后停止服务器。

启动 Tableau Server 。如果在初始化过程中未使用 –start-server 选项并且已完成配置 Tableau Server ，请使用此命令启动服务器：

``tsm start --request-timeout 900``

启动服务器后还需要添加管理员账户

``tabcmd initialuser --server 'localhost:80' --username 'admin' --password 'Sgmw5050'``

然后就可以 http://127.0.0.1 登陆服务器了

- 查看日志

``tail -f /root/.tableau/tsm/tsm.log``

``cat /root/.tableau/tsm/tsm.log | grep ERROR``

参考 
https://onlinehelp.tableau.com/current/guides/everybody-install-linux/zh-cn/everybody_admin_install_linux.htm

https://onlinehelp.tableau.com/current/server-linux/zh-cn/config_linux_apply_start.htm
