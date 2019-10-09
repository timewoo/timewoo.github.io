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

出现以下命令表示docker启动成功
![timewoo](https://timewoo.github.io/images/docker-mysql1.png)
```