# centos7常用服务安装Docker版

## 安装 Docker
安装一些必要的系统工具：

    sudo yum install -y yum-utils device-mapper-persistent-data lvm2

添加软件源信息：

    sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
更新 yum 缓存：

    sudo yum makecache fast
安装 Docker-ce：

    sudo yum -y install docker-ce
启动 Docker 后台服务：

    sudo systemctl start docker
查看 Docker 版本号：

    docker --version
使用docker命令时如果有Cannot connect to the Docker daemon at tcp://0.0.0.0:2375. Is the docker daemon running?这种情况，可能是没有启动Docker服务成功或者是需要在所有`docker`命令前加`sudo `，例如 `sudo docker images`，不用sudo使用docker方法：
```
unset DOCKER_HOST
unset DOCKER_TLS_VERIFY
unset DOCKER_TLS_PATH
docker ps
```
***
## JDK（非Docker）
**安装步骤**<br>
JDK下载地址
链接：https://pan.baidu.com/s/1eLLSPceIEPf9aSW6TMI63Q 
提取码：lki5 
```
tar zxvf jdk-8u181-linux-x64.tar.gz
vim /etc/profile
```
输入以下<br>
```
export JAVA_HOME=/root/java/jdk/jdk1.8.0_181  
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH
```
执行命令：<br>
```
source /etc/profile
java -version
```
**说明**<br>
JDK不用docker安装<br>
需要修改JAVA_HOME路径<br>

***
## Kafka
**命令**<br>
安装docker-compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose -f docker-compose-single-broker.yml up
```
**说明**<br>
下载 https://github.com/wurstmeister/kafka-docker 上传到Linux服务器，
把docker-compose-single-broker.yml中的KAFKA_ADVERTISED_HOST_NAME换成服务器公网IP，修改KAFKA_CREATE_TOPICS
单机版启动方式：
    `docker-compose -f docker-compose-single-broker.yml up`
如果提示内存不够，在[docker-compose-single-broker.yml](https://github.com/javthon/ServiceInstallation/blob/master/kafka/docker-compose-single-broker.yml)添加`KAFKA_JVM_PERFORMANCE_OPTS: " -Xmx256m -Xms256m"`<br>

**所需资源**<br>
[docker-compose-single-broker.yml](https://github.com/javthon/ServiceInstallation/blob/master/kafka/docker-compose-single-broker.yml)<br>
***
## Redis
**命令**<br>
```
docker pull redis
docker run -d -p 6380:6379 -v /root/redis.conf:/usr/local/etc/redis/redis.conf --name myredis redis redis-server /usr/local/etc/redis/redis.conf
```
**说明**<br>
-p 6380:6379：将主机的6380端口映射到容器的6379端口。:6379要和redis.conf中的对应<br>
-v /root/redis.conf:/usr/local/etc/redis/redis.conf：挂载redis.conf。<br>
密码和集群等在redis.conf中配置<br>

**所需资源**<br>
[redis.conf](https://github.com/javthon/ServiceInstallation/blob/master/redis/redis.conf)<br>

**补充**<br>
配置集群方式<br>
 1.修改主redis的配置文件<br>
&nbsp;&nbsp;&nbsp;bind 0.0.0.0<br>
&nbsp;&nbsp;&nbsp;requirepass 123456<br>
2.修改从redis的配置文件<br>
&nbsp;&nbsp;&nbsp;bind 0.0.0.0<br>
&nbsp;&nbsp;&nbsp;slaveof 192.168.1.225 6379<br>
&nbsp;&nbsp;&nbsp;masterauth 123456<br>
&nbsp;&nbsp;&nbsp;requirepass 123456<br>
3.启动redis服务<br>
&nbsp;&nbsp;先启动主redis，在启动从redis<br>

***
## MongoDB
**命令**<br>
```
docker pull mongo
docker run --name mongodb -v /root/mongo:/data/db -p 27017:27017 -d mongo --auth
docker exec -it mongodb mongo admin

use admin

db.createUser({
    user:"admin",
    pwd:"admin",
    roles:[{
        role:"root",
        db:"admin"
    }]
})

db.auth("admin", "admin")

db.createUser({
    user:"developer",
    pwd:"developer2019",
    roles:[{
        role:"dbAdmin",
        db:"testdb"
    },{
        role:"readWrite",
        db:"testdb"
    }]
})

db.auth("developer", "developer2019")

按ctrl+D退出
```
**说明**<br>
-v /root/mongo:/data/db 将主机文件夹mongo挂载到容器db文件夹，mongo文件夹用来存放数据<br>
-p 27017:27017 将主机端口27017映射到容器端口27017<br>
--auth 开启权限验证<br>
连接：mongodb://developer:developer2019@192.168.1.70:27017/testdb<br>
创建了管理员用户admin并用admin创建了用户developer，赋予developer对数据库testdb的管理和读写权限<br>

**补充**<br>
MongoDB用户权限<br>
内建的角色<br>

数据库用户角色：read、readWrite;<br>
数据库管理角色：dbAdmin、dbOwner、userAdmin；<br>
集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；<br>
备份恢复角色：backup、restore；<br>
所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase<br>
超级用户角色：root // 这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）<br>
内部角色：__system<br>

角色说明：<br>
Read：允许用户读取指定数据库<br>
readWrite：允许用户读写指定数据库<br>
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile<br>
userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户<br>
clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。<br>
readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限<br>
readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限<br>
userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限<br>
dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。<br>
root：只在admin数据库中可用。超级账号，超级权限<br>

***
## MySQL
**命令**<br>
```
docker pull mysql:5.7
docker run -p 3318:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
```
**说明**<br>
-p 3318:3306：将主机的3318端口映射到容器的3306端口。<br>
-e MYSQL_ROOT_PASSWORD=root：初始化 root 用户的密码。<br>

***
## Nginx
**命令**<br>
```
docker pull nginx
docker run -p 8085:80 --name mynginx -v /root/nginx/nginx.conf:/etc/nginx/nginx.conf -v /root/nginx/default.conf:/etc/nginx/conf.d/default.conf -v /usr/dist:/usr/dist nginx
```
**说明**<br>
-p 8085:80 将主机8085端口挂载到容器80端口，80端口要和default.conf的端口对应<br>
-v /root/nginx/nginx.conf:/etc/nginx/nginx.conf 挂载nginx.conf<br>
-v /root/nginx/default.conf:/etc/nginx/conf.d/default.conf 挂载default.conf<br>
-v /usr/dist:/usr/dist 挂载映射目录，:后面的/usr/dist要和default.conf里的root路径对应<br>

**所需资源**<br>
[default.conf](https://github.com/javthon/ServiceInstallation/blob/master/nginx/default.conf)<br>
[nginx.conf](https://github.com/javthon/ServiceInstallation/blob/master/nginx/nginx.conf)<br>

***
## ElasticSearch
**命令**<br>
```
mkdir /opt/esdata
chown -R 1000:1000 /opt/esdata
docker pull elasticsearch:6.8.0
docker run --name=myes -d -p 9200:9200 -p 9300:9300 -v /root/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /opt/esdata:/usr/share/elasticsearch/data -e "discovery.type=single-node" -e ES_JAVA_OPTS='-Xms100m -Xmx100m' elasticsearch:6.8.0
```
**说明**<br>
-p 9200:9200 将主机9200的http端口挂载到容器9200的http端口<br>
-p 9300:9300 将主机9300的tcp端口挂载到容器9300的tcp端口<br>
-v /root/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml 挂载配置文件<br>
-v /opt/data:/usr/share/elasticsearch/data 挂载数据目录<br>
-e "discovery.type=single-node" 单节点启动<br>
-e ES_JAVA_OPTS='-Xms100m -Xmx100m' 根据情况跳转Java内存大小<br>


如果出现maxvirtual memory areas vm.max_map_count [65530] istoo low, increase to at least [262144]
解决方式： 
```
vim /etc/sysctl.conf 
vm.max_map_count=655300 
sysctl -p
```

**所需资源**<br>
[elasticsearch.yml](https://github.com/javthon/ServiceInstallation/blob/master/elasticsearch/elasticsearch.yml)<br>

***

## 常用命令
* 查看所有镜像
`docker images`
* 查看正在运行的容器
`docker ps`
* 查看所有的容器
`docker ps -a`

* 容器镜像删除
1. 停止所有的container，这样才能够删除其中的images：<br>
`docker stop $(docker ps -a -q)`<br>
`docker rm $(docker ps -a -q)`

2. 查看当前有些什么images<br>
`docker images`

3. 删除images，通过image的id来指定删除谁<br>
`docker rmi <image id>`

4. 想要删除untagged images，也就是那些id为<None>的image的话可以用<br>
`docker rmi $(docker images | grep "^<none>" | awk "{print $3}")`
    
5. 要删除全部image的话<br>
`docker rmi $(docker images -q)`

* 进入容器
`docker exec -it <容器ID> /bin/bash`

***
## 卸载docker
```
sudo yum remove docker-ce
sudo rm -rf /var/lib/docker
```
如若没有完全卸载干净尝试以下方法：<br>
1.查询安装过的包<br>
```
yum list installed | grep docker
```
docker-engine.x86_64                 17.03.0.ce-1.el7.centos         @dockerrepo

2.删除安装的软件包
```
yum -y remove docker-engine.x86_64
```

## 官方镜像库
https://hub.docker.com/

## 学习资源
https://www.runoob.com/docker/docker-tutorial.html<br>
https://github.com/veggiemonk/awesome-docker<br>
http://www.docker.org.cn/<br>
https://docs.docker.com/get-started/<br>
https://www.qikqiak.com/k8s-book/docs/13.Dockerfile%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5.html<br>
https://www.cnblogs.com/dazhoushuoceshi/p/7066041.html<br>
