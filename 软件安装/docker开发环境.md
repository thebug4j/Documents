# Mac M1 docker开发环境

[参考链接](http://t.csdn.cn/dlOpU)

[TOC]



### 1、Nginx

```
#1拉取镜像
docker pull nginx:1.23.0
#2运行容器
docker run --name nginx -d -p 80:80 nginx:1.23.0
#3拷贝配置文件
docker cp nginx:/etc/nginx /Users/thebug4j/docker_data/nginx/conf
docker cp nginx:/usr/share/nginx /Users/thebug4j/docker_data/nginx/data

@4docker desktop 停止并删除刚刚创建的容器

#5创建容器 数据卷挂载
docker run --name nginx \
-v /Users/thebug4j/docker_data/nginx/conf:/etc/nginx \
-v /Users/thebug4j/docker_data/nginx/data:/usr/share/nginx \
-v /Users/thebug4j/docker_data/nginx/log:/var/log/nginx \
-p 8080:80 \
-d nginx:1.23.0

分别挂载了 配置文件、数据、日志文件


```



### 2、mysql

```
#1拉取镜像
docker pull mysql:8.0
#2运行容器
docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=longmao mysql:8.0

#3创建容器 数据卷挂载
docker run --name mysql -d -v /Users/thebug4j/docker_data/mysql/conf:/etc/mysql/conf.d \
-v /Users/thebug4j/docker_data/mysql/data:/var/lib/mysql \
-v /Users/thebug4j/docker_data/mysql/log:/var/log/mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=longmao \
mysql:8.0

分别挂载了 配置文件、数据、日志文件

tips:
mysql --help |more 可以看到配置文件的加载顺序，同样配置，可以被覆盖。
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf 

cat /etc/my.cnf 
!includedir /etc/mysql/conf.d/
由此可知，有需要更改的配置，放在/etc/mysql/conf.d/目录中即可
```

### 3、redis

```
#1拉取镜像
docker pull redis:7.0.2
#2创建配置文件
vi /Users/thebug4j/docker_data/redis/conf/redis.conf

port 6379     
bind 0.0.0.0
requirepass longmao
daemonize no                               
pidfile /var/run/redis_6379.pid
logfile "/var/log/redis.log"
dbfilename dump.rdb
dir /data/
databases 16
appendonly yes
appendfilename "appendonly.aof"
# appendfsync always
appendfsync everysec
# appendfsync no

#5创建容器 数据卷挂载
docker run --name redis -d \
-p 6379:6379 \
-v /Users/thebug4j/docker_data/redis/conf:/usr/local/etc/redis \
-v /Users/thebug4j/docker_data/redis/data:/data \
-v /Users/thebug4j/docker_data/redis/log:/var/log \
redis:7.0.2 \
redis-server /usr/local/etc/redis/redis.conf

```



### 4、rabbitmq

```
1、拉取镜像
docker pull rabbitmq:3-management
2、创建容器 数据卷挂载
docker run --name rabbitmq \
-p 5672:5672 -p 15672:15672 \
-v /Users/thebug4j/docker_data/rabbitmq/data:/var/lib/rabbitmq/mnesia \
-v /Users/thebug4j/docker_data/rabbitmq/log:/var/log/rabbitmq/log \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin  \
-d rabbitmq:3-management

2、下载对应版本的延迟插件
https://github.com/rabbitmq/rabbitmq-delayed-message-exchange

3、上传插件到容器内部
docker cp /root/rabbitmq_delayed_message_exchange-3.10.2.ez rabbitmq:/plugins

4、进入容器内部开启延迟插件
rabbitmq-plugins enable rabbitmq_delayed_message_exchange

5、退出容器重启服务
docker restart rabbitmq

```



### 5、Elastic Search

[Es 和 kibana 参考](http://t.csdn.cn/06PFy)

#### Elastic

```
#1、拉取镜像
docker pull elasticsearch:8.3.2
#2、创建网络
docker network create mynetwork
#3、运行容器
docker run --name elasticsearch -d \
-p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms200M -Xmx200M" \
--net mynetwork \
elasticsearch:8.3.2

#4拷贝配置文件
docker cp elasticsearch:/usr/share/elasticsearch/config /Users/thebug4j/docker_data/elasticsearch/config/

docker cp elasticsearch:/usr/share/elasticsearch/data /Users/thebug4j/docker_data/elasticsearch/data/

docker cp elasticsearch:/usr/share/elasticsearch/plugins /Users/thebug4j/docker_data/elasticsearch/plugins/

@4docker desktop 停止并删除刚刚创建的容器

#5创建容器 数据卷挂载
docker run --name elasticsearch -d \
-p 9200:9200 -p 9300:9300 \
-v /Users/thebug4j/docker_data/elasticsearch/config:/usr/share/elasticsearch/config \
-v /Users/thebug4j/docker_data/elasticsearch/data:/usr/share/elasticsearch/data \
-v /Users/thebug4j/docker_data/elasticsearch/plugins:/usr/share/elasticsearch/plugins  \
-v /Users/thebug4j/docker_data/elasticsearch/logs:/usr/share/elasticsearch/logs  \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms200M -Xmx200M" \
--net mynetwork \
elasticsearch:8.3.2

#6进入交互界面设置密码
./bin/elasticsearch-setup-passwords interactive

tips：跨域问题
config/elasticsearch.yml中添加：
http.cors.enabled: true
http.cors.allow-origin: "*"


```

#### kibana

```
#1、拉取镜像
docker pull kibana:8.3.2

#2、运行容器
docker run -d --name kibana \
-p 5601:5601 \
-e ELASTICSEARCH_URL=http://elasticsearch:9200 \
--net mynetwork \
kibana:8.3.2

#3拷贝配置文件
docker cp kibana:/usr/share/kibana/config/ /Users/thebug4j/docker_data/kibana/config

@4删除容器

#5创建容器 数据卷挂载
docker run -d --name kibana \
-v /Users/thebug4j/docker_data/kibana/config:/usr/share/kibana/config \
-v /Users/thebug4j/docker_data/kibana/data:/usr/share/kibana/data \
-p 5601:5601 \
-e ELASTICSEARCH_URL=http://elasticsearch:9200 \
--net mynetwork \
kibana:8.3.2

#6登陆页面会要求输入token
到es容器内，执行
bin/elasticsearch-create-enrollment-token --scope kibana
拷贝并粘贴token，认证通过后输入账号密码即可登陆。elastic/elastic

#7 如何切换中文
在config/kibana.yml添加
i18n.locale: "zh-CN" 

```

#### ik分词器

```
#1 拷贝ik分词器插件到对应的es服务器挂载目录并解压
/Users/thebug4j/Downloads/安装包/elasticsearch-analysis-ik-8.3.2.zip
/Users/thebug4j/docker_data/elasticsearch/plugins

#2 登录kibana测试ik分词器是否生效

#ik_smart 智能切分，粗粒度
POST /_analyze
{
  "text": "周杰伦喜欢唱青花瓷",
  "analyzer": "ik_smart"
}

# ik_max_word 最细切分，细粒度
POST /_analyze
{
  "text": "周杰伦喜欢唱青花瓷",
  "analyzer": "ik_max_word"
}

```



### 6、zookeeper

```
#1、拉取镜像
docker pull zookeeper:3.8.0
#2、运行容器
docker run --name zookeeper -d -p 2181:2181 -p 2888:2888 -p 3888:3888 zookeeper:3.8.0
#3拷贝配置文件
mkdir -p /Users/thebug4j/docker_data/zookeeper/conf/
docker cp zookeeper:/conf/zoo.cfg /Users/thebug4j/docker_data/zookeeper/conf/
@4删除容器

#5、创建容器
docker run --name zookeeper -d \
-v /Users/thebug4j/docker_data/zookeeper/conf:/conf \
-v /Users/thebug4j/docker_data/zookeeper/data:/data \
-v /Users/thebug4j/docker_data/zookeeper/log:/datalog \
-p 2181:2181 -p 2888:2888 -p 3888:3888 \
zookeeper:3.8.0

```

### 7、centos

```
#1、拉镜像
docker pull centos:7
#2、运行容器
docker run -d -p 2222:22 --name centos7 --privileged=true centos:7 /usr/sbin/init
#3、进入交互界面，安装必须的包
yum install net-tools openssh-server vim lrzsz wget gcc-c++ pcre pcre-devel zlib zlib-devel ruby openssl openssl-devel patch bash-completion zlib.i686 libstdc++.i686 lsof unzip zip openssh-server -y
#4、设置root密码（liujianlong）
passwd root


#5、开启远程登录
ssh-keygen -t ecdsa -P '' -f /etc/ssh/ssh_host_ecdsa_key
ssh-keygen -t ed25519 -P '' -f /etc/ssh/ssh_host_ed25519_key
/usr/sbin/sshd
@6、ssh工具进行登录 2222端口

```



### 8、nacos

```
#1、拉镜像并运行容器
docker run \
--name nacos-server \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
-e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=192.168.31.121 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_USER=nacos \
-e MYSQL_SERVICE_PASSWORD=nacos \
-e MYSQL_SERVICE_DB_NAME=nacos \
-e TIME_ZONE='Asia/Shanghai' \
-d nacos/nacos-server:v2.1.2-slim

#2、拷贝配置文件
mkdir -p /Users/thebug4j/docker_data/nacos
docker cp -a nacos-server:/home/nacos /Users/thebug4j/docker_data/nacos

@3、关闭并删除该容器

#4、重新运行容器
docker run \
--name nacos-server \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
-e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=1.117.161.106 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=Liu@Xmy520 \
-e MYSQL_SERVICE_DB_NAME=nacos2 \
-e TIME_ZONE='Asia/Shanghai' \
-v /Users/thebug4j/docker_data/nacos/nacos/conf:/home/nacos/conf \
-v /Users/thebug4j/docker_data/nacos/nacos/logs:/home/nacos/logs \
-v /Users/thebug4j/docker_data/nacos/nacos/data:/home/nacos/data \
-d nacos/nacos-server:v2.1.2-slim


```

### 9、chatgpt-on-wechat

```
#1、拉镜像并运行容器
docker pull zhayujie/chatgpt-on-wechat

#2、运行
docker run \
--name chatgpt-on-wechat \
-d zhayujie/chatgpt-on-wechat:latest

#3、拷贝配置文件
mkdir -p /Users/thebug4j/docker_data/chatgpt-on-wechat
docker cp -a chatgpt-on-wechat:/app/config.json /Users/thebug4j/docker_data/chatgpt-on-wechat/config.json

@4、关闭并删除该容器

#4、重新运行容器
docker run \
--name chatgpt-on-wechat \
-v /Users/thebug4j/docker_data/chatgpt-on-wechat/config.json:/app/config.json \
-d zhayujie/chatgpt-on-wechat:latest

```

