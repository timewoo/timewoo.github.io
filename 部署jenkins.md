## docker部署jenkins(centos7)
记录部署jenkins，实现自动化部署的操作步骤

### 部署jenkins
```
###docker拉取jenkins镜像
docker pull jenkins/jenkins

###创建挂载目录
mkdir -p /home/data/jenkins

###给挂载目录赋权限
chown -R 1000:1000 /home/data/jenkins
```
如果没有设置挂载目录的权限，启动jenkins会出现下面错误
![timewoo](https://timewoo.github.io/images/jenkins1.png)
因为外部挂载目录的用户是root，设置挂载目录之后访问/var/jenkins_home的拥有者变成了root，导致jenkins容器操作/var/jenkins_home没有权限，解决办法就是把挂载目录同时赋权给jenkins容器，jenkins容器的uid是1000。

```
###启动jenkins
docker run -p 8002:8080 -p 50000:50000 \
	-v /home/data/jenkins/:/var/jenkins_home \
	--name jenkins -e JAVA_OPTS=-Duser.timezone=Asia/Shanghai --privileged=true --restart=always -d jenkins
```

访问服务器8002端口即可跳转到登录页面
![timewoo](https://timewoo.github.io/images/jenkins2.png)

查看容器日志获取初始登录密码
![timewoo](https://timewoo.github.io/images/jenkins3.png)

登录成功选择jenkins插件安装方式，默认选择第一种
![timewoo](https://timewoo.github.io/images/jenkins4.png)

由于jenkins默认使用国外的镜像源下载，速度很慢，所以更改数据源之后在下载相关插件

设置jenkins的登录用户
![timewoo](https://timewoo.github.io/images/jenkins5.png)

登录完成后打开【系统管理】--【插件管理】--【高级】设置插件源地址，重新下载插件
![timewoo](https://timewoo.github.io/images/jenkins6.png)
![timewoo](https://timewoo.github.io/images/jenkins7.png)
![timewoo](https://timewoo.github.io/images/jenkins8.png)
推荐镜像源为https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json