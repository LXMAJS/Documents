## 初始化一台全新的Linux服务器

本文主要讲述如何在一台全新的Linux服务器（系统CentOS）上搭建软件和服务，软件的优先级从上至下

### 一、Docker

docker作为轻量级应用容器的载体，在你的软件应用初期能给你提供一个很好的容器管理、CI/CD 等运维功能，也能让你很快的将软件服务启动起来，下面将按照实际需要依次讲解 docker 和 docker-compose 的安装。

#### 1、docker

参考：[Linux安装Docker完整教程](https://blog.csdn.net/m0_59196543/article/details/124749175)

##### a. 检查

执行以下命令来检查你的环境是否安装了docker，如果安装好了，可以执行 `` docker -v `` 来检查docker的版本。

``` shell
yum list installed | grep docker
```

安装成功后将会是下面这样的：

![image-20230526154126934](/Users/lxmajs/Library/Application Support/typora-user-images/image-20230526154126934.png)

##### b. 安装

执行下面的命令来安装docker

``` shell
yum -y install docker-ce
```

![image-20230526154345642](/Users/lxmajs/Library/Application Support/typora-user-images/image-20230526154345642.png)

安装好以后，可以看到 “Complete!” 的字样，执行上一步的命令，检查是否已经安装好docker。

##### c. 启动

执行下面的命令，可以立即运行docker：

``` shell
# 启动docker
systemctl start docker
# 停止docker
systemctl stop docker
```

如果希望服务器启动或被重启时，自动运行docker，可以执行下面的命令：

``` shell
systemctl enable docker
```

启动后，如果要检查docker运行的状态，可以执行下面的命令：

``` shell
systemctl status docker
```

##### d. 镜像仓库配置

docker官方的镜像网速较差，我们最好设置为国内的镜像仓库。依次执行下面的命令：

1）创建文件夹

``` shell
mkdir -p /etc/docker
```

2）在文件夹内新建一个daemon.json文件 

``` shell
sudo tee /etc/docker/daemon.json << -'EOF' 
{  
	"registry-mirrors": ["https://akchsmlh.mirror.aliyuncs.com"] 
} 
EOF
```

或进入到该目录，执行 vim daemon.json 后插入以下内容：

``` json
{
  "registry-mirrors": ["https://akchsmlh.mirror.aliyuncs.com"]
}
```

3）重启docker

``` shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 2、docker-compose

docker-compose是docker的一个插件工具，可以提供以 docker-compose.yaml 文件形式书写的配置文件，并执行 `` docker-compose up -d `` 来运行docker容器的功能。我们在日常维护docker容器时，如果一个软件有较多的容器需要按顺序依次启动，用 docker-compose.yaml 文件来维护容器配置是十分好的选择。

##### a. 安装

安装docker-compose插件工具是需要从github上选择安装的，这里给一个 1.21.2 的版本的下载方式，并默认将其下载到系统指令集的目录下：

``` shell
curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

![image-20230526154840875](/Users/lxmajs/Library/Application Support/typora-user-images/image-20230526154840875.png)

下载好以后，需要给 docker-compose 命令授权

##### b. 授权

执行下面的命令来给 docke-compose 授权

``` shell
# 进入命令集目录
cd /usr/local/bin/
# 检查docker-compose是否已下载
ls -l | grep docker-compose

# 授权
chmod +x /usr/local/bin/docker-compose
```

![image-20230526155206686](/Users/lxmajs/Library/Application Support/typora-user-images/image-20230526155206686.png)

可以看到授权前是没有可执行权限的，下面就可以执行 `` docker-compose -v `` 来查看版本号了

### 二、数据库（Mysql）

我们使用的数据库一般是mysql和elasticsearch，这里先说Mysql，安装的方式用docker的方式来部署

#### 1、安装

我们使用 5.7.30 的数据库来举例

##### a. 拉取镜像

``` shell
# 搜索镜像（非必需）
docker search mysql:5.7.30

# 拉取镜像
docker pull mysql:5.7.30
```

##### b. 配置 docker-compose.yaml

这里需要配置一个docker-compose.yaml文件，来设置一些docker的配置信息，当然你也可以直接使用 docker run 的方式来跑去容器，只不过有很多配置我们需要另外设置。所以推荐用 docker-compose

``` yaml
version: '2'
services:
  mysql:
    image: mysql:5.7.30
    container_name: mysql-5.7.30
    network_mode: host
    environment:
      MYSQL_ROOT_PASSWORD: 123456 # 这里是默认的数据库密码，你也可以改为别的
      TZ: Asia/Shanghai
      LANG: C.UTF-8
    ports:
      - 3306:3306
    volumes:
      - /data/mysql-5730/data:/var/lib/mysql
      - /data/mysql-5730/log:/var/log/mysql
      - './mysql.cnf:/etc/mysql/conf.d/mysql.cnf'
      - '/etc/localtime:/etc/localtime'
    restart: unless-stopped
```

直接拷贝上面的内容生成一个 docker-compose.yaml 文件即可

##### c. 配置 mysql.conf

``` conf
[mysqld]
sql-mode="NO_ENGINE_SUBSTITUTION"
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
skip-name-resolve
transaction-isolation = READ-COMMITTED
character-set-server=utf8mb4
character-set-client-handshake=FALSE
init_connect='SET NAMES utf8mb4'
collation-server=utf8mb4_unicode_ci

max_connections = 1500
max_allowed_packet = 500M

slow_query_log=1
slow_query_log_file=/var/log/mysql/mysqld-slow.log
long_query_time=5

symbolic-links = 0

tmp_table_size = 256M
max_heap_table_size = 256M
join_buffer_size = 256M
innodb_buffer_pool_size = 4G
innodb_buffer_pool_instances = 2
group_concat_max_len = 102400

[mysqld_safe]
log-error=/var/log/mysql/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[client]
default-character-set=utf8mb4
socket=/var/lib/mysql/mysql.sock

[mysql]
default-character-set=utf8mb4
```

##### d. 验证启动

在docker-compose所在的目录直接执行下面的命令即可：

``` shell
docker-compose up -d
```

如果配置有修改，或者需要重启，也可以直接上面的命令，不需要停止容器再执行。

##### e. 允许远程连接

> 首先确认你的服务器的安全组或防火墙已经将 3306（或你自定义的端口）暴露，并允许访问。

执行下面的几行命令来操作：

``` shell
# 登录mysql
mysql -h127.0.0.1 -uroot -pxxx

# 修改密码（非必需）
alter user 'root'@'localhost' identified by '#这里是你的密码#';

# 授权
grant all privileges on *.* to 'root'@'%' identified by '#这里是你的密码#';

# 刷新权限缓存
flush privileges;
```

看到提示OK后表示成功，此时可以用客户端工具来远程链接了！

![image-20230526161155768](/Users/lxmajs/Library/Application Support/typora-user-images/image-20230526161155768.png)



#### 2、分配权限

