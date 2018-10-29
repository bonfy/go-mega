# 15-Deployment On Linux

在本章，我将会把应用部署到Linux服务器, 与上一章相比，这种是比较传统的方式

_本章的GitHub链接为：_ [Source](https://github.com/bonfy/go-mega-code/tree/15-Deployment-On-Linux), [Diff](https://github.com/bonfy/go-mega-code/compare/14-Deployment-On-Heroku...15-Deployment-On-Linux), [Zip](https://github.com/bonfy/go-mega-code/archive/v1.5.zip)

## Vultr

当提到“传统托管”时，意思是应用是手动或通过原始服务器机器上的脚本安装部署的。 该过程涉及安装应用程序、其依赖项和生产规模的Web服务器，并配置系统以确保其安全。

当你要部署自己的项目时，要问的第一个问题是在哪找服务器。 目前有很多经济的托管服务。 只需每月2.5 - 5美元，[Vultr](https://www.vultr.com/),[Digital Ocean](https://www.digitalocean.com/)和[Linode](https://www.linode.com/)

这里我选择的是 Vultr 创建了 CentOS 7 的linux 服务器

> 广告: 这里大家可以通过这个地址注册 Vultr，我会得到 10 dollar 的奖励，如果想搞个VPS自己玩玩的同学可以考虑下，还能搭个那啥用(自己体会) 链接: [https://www.vultr.com/?ref=7079936](https://www.vultr.com/?ref=7079936)


## ssh 免密登陆

```cmd
$ ssh-keygen -t rsa -C  'your email'

$ scp ~/.ssh/id_rsa.pub username@hostname:~/ # 将公钥文件复制至ssh服务器
$ ssh username@hostname # 使用用户名和密码方式登录至ssh服务器
$ mkdir .ssh  # 若.ssh目录已存在，可省略此步
$ cat id_rsa.pub >> .ssh/authorized_keys  # 将公钥文件id_rsa.pub文件内容追加到authorized_keys文件
```

以后登录ssh 只要输入 `ssh username@hostname` 即可不用输入密码啦

## 安装mysql

```cmd
$ ssh username@hostname

# Install
$ yum -y install mariadb mariadb-serve

# Start mariadb
$ sudo systemctl start mariadb
$ sudo systemctl enable mariadb

# Securing the MariaDB Server
# Set Password
$ sudo mysql_secure_installation

# crate database
$ mysql -uroot -p
$ MariaDB > create database go-mega;
```

## 部署应用

与 Python 相比，Go 部署应用还是算比较方便的，因为它可以编译

### Makefile

为了方便部署，我们这边建一个  `Makefile` 方便我们部署

Makefile
```
.PHONY: clean upload-vps

# Put your server ip or nickname here
host = ny

clean:
	@echo "clean..."
	@rm -rf app/
	@echo "success clean"
build-db:
	@echo "run build db..."
	@env GOOS=linux go build -o app/db cmd/db_init/main.go
	@echo "success build db"
build-server:
	@echo "begin build server..."	
	@env GOOS=linux go build -o app/server main.go
	@echo "success build server"
upload-vps: clean build-db build-server
	@echo "begin run vps upload..."
	@echo "Host: $(host)"
	@scp -r app root@$(host):go-mega
	@scp config.yml root@$(host):go-mega/
	@scp -r templates root@$(host):go-mega/templates
	@echo "success vps upload"
```

注意 上面的 Host 修改成 你自己的 Host ip address 或者 Host 别名

然后我们就比较方便了运行

```cmd
$ make upload-vps
```

就可以了，我们可以登录到我们的vps上查看，就多了一个 go-mega 的文件夹了

这里简要做个说明

* clean: 清楚上次的编译二进制文件
* build-db: 编译 cmd/db_init/main.go -> app/db
* build-server:  编译 main.go -> app/server
* upload-vps: 将 /app 和 templates 文件夹上传到 vps的 ~/go-mega 里

### 运行

```cmd
$ ssh ssh username@hostname

$ cd go-mega

# 设置环境变量 以及 config.yml 
$ vi config.yml # 按照config.yml.sample 设置
$ export PORT=8000

# 初始化数据库
$ ./db
# 启动应用
$ nohup ./server &

# 这个时候服务已经启动了，这个时候如果想要访问应用可以使用 nginx反代到 80端口 或者 防火墙开放你设置的端口
# 最好是使用nginx，不过这里为了演示，我们开放下端口
$ firewall-cmd --zone=public --add-port=8000/tcp
```

这个时候我们访问 `http://xxx.xxx.xxx.xxx:8000` 就可以访问你刚才发布的应用了, 大功告成

> 这里暂时不是HTTPS的，如果设置HTTPS，需要你自己申请HTTPS证书和配置nginx，这方面比 Heroku 还是麻烦点

## Links

  * [目录](README.md)
  * 上一节: [14-Deployment-On-Heroku](14-deployment-on-heroku.md)
  * 下一节: [16](16-summary.md)