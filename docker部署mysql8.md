## docker部署mysql8(centos7)
记录自己在centos7上部署mysql8的步骤，主要熟悉docker的安装命令和mysql8的新特性
### docker部署

```markdown
centos7安装docker相关命令

###安装docker service
yum install docker -y

###修改docker仓库的镜像地址，采用阿里的镜像地址
vim /etc/docker/daemon.json
{
  "registry-mirrors":["https://nxkiihz2.mirror.aliyuncs.com"]
}

###设置docker开机自启
systemctl enable docker   开机自启
systemctl start docker    启动docker

###测试docker服务是否启动
docker run hello-world

```
出现以下命令表示docker启动成功  
![timewoo](https://timewoo.github.io/images/docker-mysql1.png)

### docker部署mysql8
两种方式拉取mysql8的镜像
1.直接用search命令查询镜像
2.dockerhub直接查询，好处是可以查询镜像的版本
直接使用search
```markdown
docker search mysql

###拉取镜像
docker pull docker.io/mysql

###查看镜像是否拉取成功
docker images

###创建mysql的数据的存放目录(必须挂载在容器外，不然重启容器数据会消失)
mkdir -p /home/data/mysql/data

###创建mysql的log日志地址
mkdir -p /home/data/mysql/log

###创建数据库配置文件
touch /home/data/mysql/config/my.cnf
```
my.cnf的配置如下
```markdown
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Custom config should go here
!includedir /etc/mysql/conf.d/

default_authentication_plugin= mysql_native_password
```

```markdown
###设置挂载目录和配置文件之后需要添加mysql-files，否则无法启动
mkdir -p /home/data/mysql/mysql-files

###启动mysql
docker run \
	-p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 \
	-v /home/data/mysql/data:/var/lib/mysql:rw \
	-v /home/data/mysql/log:/var/log/mysql:rw \
	-v /home/data/mysql/mysql-files:/var/lib/mysql-files:rw \
	-v /home/data/mysql/config/my.cnf:/etc/mysql/my.cnf:rw \
	-v /etc/localtime:/etc/localtime:ro \
	--name mysql8 --restart=always --privileged=true -d mysql

###/etc/localtime:/etc/localtime:ro 是让容器的时钟与宿主机时钟同步，避免时区的问题，ro是read only的意思，就是只读
###-p 设置对外端口，-v 设置挂载目录，--name 设置别名，--restart=always 设置开机自启，--privileged=true 设置root权限 
###-d 后台运行

###查看mysql运行
docker ps

###进入容器查看
docker exec -it mysql8 bash
mysql -uroot -p123456
mysql启动成功
```
###mysql8客户端连接问题
由于mysql8更新了密码加密算法，如果没有提前修改密码算法，会导致mysql客户端无法连接，可以使用docker进入mysql容器修改算法
```markdown
mysql> ALTER user 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
Query OK, 0 rows affected (0.11 sec)

mysql> ALTER user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.11 sec)

mysql>  FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)
```
