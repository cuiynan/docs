[TOC]

# 一、centeros

## vm 安装后无法上网

> #更改  ONBOOT=yes
>
> vim /etc/sysconfig/network-scripts/ifcfg-ens33    
>
> service network restart

## 安装docker

 ```shell
 yum install -y yum-utils device-mapper-persistent-data lvm2
 yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
 yum list docker-ce --showduplicates | sort -r
 yum install docker-ce
 systemctl start docker
 systemctl enable docker
 ```

### docker 安装mysql

新建下方文件

/mysql/Dockerfile

> FROM mysql:5.7
>
> #设置免密登录
> ENV MYSQL_ALLOW_EMPTY_PASSWORD yes
>
> #将所需文件放到容器中
> COPY setup.sh /mysql/setup.sh
> COPY schema.sql /mysql/schema.sql
> COPY privileges.sql /mysql/privileges.sql
>
> #设置容器启动时执行的命令
> CMD ["sh", "/mysql/setup.sh"]

/mysql/setup.sh

> #!/bin/bash
> set -e
>
> #查看mysql服务的状态，方便调试，这条语句可以删除
> echo `service mysql status`
>
> echo '1.启动mysql....'
> #启动mysql
> service mysql start
> sleep 3
> echo `service mysql status`
>
> echo '2.开始导入数据....'
> #导入数据
> mysql < /mysql/schema.sql
> echo '3.导入数据完毕....'
>
> sleep 3
> echo `service mysql status`
>
> #重新设置mysql密码
> echo '4.开始修改密码....'
> mysql < /mysql/privileges.sql
> echo '5.修改密码完毕....'
>
> #sleep 3
> echo `service mysql status`
> echo `mysql容器启动完毕,且数据导入成功`
>
> tail -f /dev/null

/data/schema.sql

> -- 创建数据库
> create database `docker_mysql` default character set utf8 collate utf8_general_ci;
>
> use docker_mysql;
>
> -- 建表
> DROP TABLE IF EXISTS `user`;
>
> CREATE TABLE `user` (
>  `id` bigint(20) NOT NULL,
>  `created_at` bigint(40) DEFAULT NULL,
>  `last_modified` bigint(40) DEFAULT NULL,
>  `email` varchar(255) DEFAULT NULL,
>  `first_name` varchar(255) DEFAULT NULL,
>  `last_name` varchar(255) DEFAULT NULL,
>  `username` varchar(255) DEFAULT NULL,
>  PRIMARY KEY (`id`)
> ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
>
> -- 插入数据
> INSERT INTO `user` (`id`, `created_at`, `last_modified`, `email`, `first_name`, `last_name`, `username`)
> VALUES
>   (0,1490257904,1490257904,'john.doe@example.com','John','Doe','user');

privileges.sql

>use mysql;
>select host, user from user;
>-- 因为mysql版本是5.7，因此新建用户为如下命令：
>create user docker identified by '123456';
>-- 将docker_mysql数据库的权限授权给创建的docker用户，密码为123456：
>grant all on docker_mysql.* to docker@'%' identified by '123456' with grant option;
>-- 这一条命令一定要有：
>flush privileges;


```shell
docker build -t 13/docker-mysql .
docker exec -ti xxx sh
```

> 参考https://www.jb51.net/article/115422.htm