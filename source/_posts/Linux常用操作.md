---
title: Linux常用操作
date: 2019-12-19 15:50:00
categories: Linux
tags: [Java,Linux]
---

## Linux 常用操作

### 防火墙开启端口

#### centos6

* 开启相关端口

> /sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT

> /sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT

<!-- more --> 

* 保存配置

> /etc/rc.d/init.d/iptables save

* 重启服务

> /etc/rc.d/init.d/iptables restart

#### centos7

* 放开防火墙8888端口

```
firewall-cmd --permanent --zone=public --add-port=9090/tcp
```

* 重启防火墙

```
firewall-cmd --reload
```

### 卸载mysql

* 源码安装

```
find / -name mysql
rm -rf 上边查找到的路径，多个路径用空格隔开
#或者下边一条命令即可
find / -name mysql|xargs rm -rf
```

* rpm安装

```
// 查看
rpm -qa | grep mysql
// 删除
rpm -e mysql　　// 普通删除模式
rpm -e --nodeps mysql　　// 强力删除模式，如果使用上面命令删除时，提示有依赖的其它文件，则用该命令可以对其进行强力删除
```

### scp免密传输文件

* 生成配对密钥


  ```
  执行命令：ssh-keygen -t rsa
  过程：一路enter不要输入密码
  /root/.ssh/id_rsa already exists.
  Overwrite (y/n)? y
  Enter passphrase (empty for no passphrase): 
  Enter same passphrase again: 
  Your identification has been saved in /root/.ssh/id_rsa.
  Your public key has been saved in /root/.ssh/id_rsa.pub.
  The key fingerprint is:
  SHA256:Izal7G5mpRxGiJ+mMHVZO4keVZuCrqGrHB8z2Qz7E8A root@localhost.localdomain
  The key's randomart image is:
  ```

* 将`/root/.ssh`文件夹下的`id_rsa.pub`上传到目标服务器的相同路径并重命名为`authorized_keys`


  ```
  scp id_rsa.pub root@47.100.168.214:/root/.ssh/authorized_keys
  ```

### 搜索文件

* find命令


  ```
  find / -name 'findname'
  搜索根目录下所有名为findname的文件
  ```

* locate命令


  ```
  locate 'findname'
  ```

### Linux查看日志

* 查看日志： 

```
常用，显示最后3000行：
cat api.log | tail -n 3000

linux 如何显示一个文件的某几行(中间几行)

从第3000行开始，显示1000行。即显示3000~3999行
cat filename | tail -n +3000 | head -n 1000

显示1000行到3000行
cat filename| head -n 3000 | tail -n +1000

*注意两种方法的顺序
分解：
tail -n 1000：显示最后1000行
tail -n +1000：从1000行开始显示，显示1000行以后的
head -n 1000：显示前面1000行

用sed命令
sed -n '5,10p' filename 这样你就可以只查看文件的第5行到第10行。

例：cat mylog.log | tail -n 1000 #输出mylog.log 文件最后一千行
```

* 查找指定信息

```
cat log.txt | grep 'ERROR' -A 5

意思是，在log.txt文件中，查找ERROR字符，并显示ERROR所在行的之后5行

cat log.txt | grep 'ERROR' -B 5  之前5行

cat log.txt | grep 'ERROR' -C 5 前后5行

cat log.txt | grep -v 'ERROR' 排除ERROR所在的行

```

### linux公钥免密码登录

```
ssh-keygen -t rsa获得id_rsa、id_rsa.pub
将.pub文件中公钥复制到服务器的/root/.ssh文件夹中的authorized_keys文件中

```

## nginx常用操作

> 待完善


## 别名启动.sh脚本

* 1、~路径下新增文件

```
touch .bash_profile
```
* 2、编辑
  
```
vi .bash_profile
```

* 3、写入

```
# ssh管理脚本
alias so=/Users/tang/source/so/so.sh
```

## scp命令
```
从服务器下载整个文件夹到指定服务器文件夹
scp -r root@47.101.184.179:/home/mins-test/cloud/qrcode Downloads/
```

## 查看最近登录的几个用户

```
last -n 100|awk '{print $1"\t"$3}'
```