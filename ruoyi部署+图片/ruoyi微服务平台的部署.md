实验报告: ruoyi微服务平台的部署

| 实验名称 | ruoyi微服务平台的部署  | 班级   | 第四组      |     |
| ---- | -------------- | ---- | -------- | --- |
| 学生姓名 | 仝光耀            | 实验日期 | 2026-6-2 |     |
| 操作系统 | openEuler24.03 | 部署方式 |          |     |
|      |                |      |          |     |
## **一. 基本信息**

| 实验名称  | ruoyi微服务平台的部署 | 班级/组别   | 第四组      |
| ----- | ------------- | ------- | -------- |
| 当前负责人 | 仝光耀           | 实验/交付日期 | 2026-6-2 |
## **二. 实验概述与目的**
**实验的目的：**
掌握容器化部署技能学习使用 Docker 容器化技术部署企业级微服务系统,理解微服务架构认识服务拆分、服务注册发现、配置中心等核心概念,熟悉主流技术栈Spring Cloud + Nacos + MySQL + Redis + Nginx 等技术整合,培养运维能力掌握镜像构建、容器编排、服务编排、日志排查等实操技能.
**系统简介:**
ruoyi是一套基于 Spring Cloud 的开源微服务权限管理系统：
- 前端：Vue.js + Element UI
- 后端：Spring Cloud Alibaba
- 注册中心/配置中心：Nacos
- 数据库：MySQL 5.7/8.0
- 缓存：Redis
- 网关：Spring Cloud Gateway

## 三. 实验环境准备
准备openEuler24.03虚拟机，2C4G40G（建议4C8G50G）

## 四. 部署与安装步骤

##### 1.系统模块

```
com.ruoyi

├── ruoyi-ui // 前端框架 [80]
├── ruoyi-gateway // 网关模块 [8080]
├── ruoyi-auth // 认证中心 [9200]
├── ruoyi-api // 接口模块
│ └── ruoyi-api-system // 系统接口
├── ruoyi-common // 通用模块
│ └── ruoyi-common-core // 核心模块
│ └── ruoyi-common-datascope // 权限范围
│ └── ruoyi-common-datasource // 多数据源
│ └── ruoyi-common-log // 日志记录
│ └── ruoyi-common-redis // 缓存服务
│ └── ruoyi-common-seata // 分布式事务
│ └── ruoyi-common-security // 安全模块
│ └── ruoyi-common-sensitive // 数据脱敏
│ └── ruoyi-common-swagger // 系统接口
├── ruoyi-modules // 业务模块
│ └── ruoyi-system // 系统模块 [9201]
│ └── ruoyi-gen // 代码生成 [9202]
│ └── ruoyi-job // 定时任务 [9203]
│ └── ruoyi-``file // 文件服务 [9300]
├── ruoyi-visual // 图形化管理模块
│ └── ruoyi-visual-monitor // 监控中心 [9100]├──pom.xml // 公共依赖
```

架构图
![](ruoyi-image/Pasted_image_20260529140747.png)
#### 1 . 首先安装
```
docker docker-compose openjdk apache-maven
```

进入到 /src/app/tools/目录下 将需要用到的软件包传到这个目录下

```
cd /src/app/tools/
scp -r test@10.203.41.22:/tmp/apache-maven-3.9.6.tgz .
scp -r test@10.203.41.22:/tmp/graalvm-jdk-21.0.4+8.1.tgz .
scp -r test@10.203.41.22:/node-v16.20.2-linux-x64.tgz .
```

#### 2 . 在当前目录下解压

```
tar -zxf apache-maven-3.9.6.tgz
tar -zxf graalvm-jdk-21.0.4+8.1.tgz
tar -zxf node-v16.20.2-linux-x64.tgz
ls
```
#### 3 .配置环境变量（wiki上的）

```
/etc/profile
export JAVA_HOME=``/root/graalvm-jdk-21``.0.4+8.1
export MAVEN_HOME=``/srv/app/tools/apache-maven-3``.9.6
export NODE_HOME=``/srv/app/tools/node-v12``.14.0-linux-x64
export PATH=``/usr/local/sbin``:``/usr/local/bin``:``/usr/sbin``:``/usr/bin``:``/
root/bin``:$JAVA_HOME``/bin``:``/usr/local/node/bin``:$MAVEN_HOME``/bin``:$NODE_HOME``/bin
source /etc/profile
```

以下是经过加以修改后的

```
export JAVA_HOME=/srv/app/tools/graalvm-jdk-21.0.4+8.1 export
MAVEN_HOME=/srv/app/tools/apache-maven-3.9.6 export NODE_HOME=/srv/app/tools/node
v16.20.2-linux-x64 export
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:$JAVA_HOME/bin:/usr/local/node/bin:$MAVEN_HOME/bin:$NODE_HOME/bin
source /etc/profile
```

或者进⼊ vim /etc/profile 这个⽂件中加⼊以下内容

```
export JAVA_HOME=/srv/app/tools/graalvm-jdk-21.0.4+8.1 export
MAVEN_HOME=/srv/app/tools/apache-maven-3.9.6 export NODE_HOME=/srv/app/tools/node-v16.20.2-linux-x64 export
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:$JAVA_HOME/bin:/usr/local/node/bin:$MAVEN_HOME/bin:$NODE_HOME/bin
```

保存退出后执⾏下⾯这条命令

```
source /etc/profile （保存刷新）
```

#### 4 .检测⼀下版本

```
docker --version
docker-compose --version
java --version
mvn -v
node -v
```
![](ruoyi-image/Pasted_image_20260529141224.png)
#### 5 .拖包RuoYi-Cloud-v3.6.6tar 在x-shell中前提需要安装⼀个⼯具
```
yum -y install lrzsz
```
![](ruoyi-image/Pasted_image_20260529141247.png)
在进⾏拖包
![](ruoyi-image/Pasted_image_20260529141259.png)
```
6 .cd /RuoYi-Cloud-v3.6.6/docker/ #### 修改docker-compose.yml
```

修改之先进⾏备份

```
cp copy.sh copy.sh.bak

cp docker-compose.yml docker-compose.yml.bak
```
![](ruoyi-image/Pasted_image_20260529141331.png)
#### 7. 把mysql、redis、nacos、nginx镜像地址全部修改为本地仓库

```
image: 10.203.41.20:8088``/pt1/nacos/nacos-server``:v2.3.2
image: 10.203.41.20:8088``/pt1/mysql``:5.7
image: 10.203.41.20:8088``/pt1/redis``:7.0.4
image: 10.203.41.20:8088``/pt1/nginx找到ruoyi-nginx 

把如下注释掉，并且保存退出
depends_on:
- ruoyi-gateway
links:
- ruoyi-gateway
```
![](ruoyi-image/Pasted_image_20260529141447.png)
![](ruoyi-image/Pasted_image_20260529141517.png)
#### 8 .  Mysql部署
```
进入RuoYi-Cloud/docker/mysql/
修改mysql dockerfile
sed -i 's/FROM mysql:5.7/FROM 10.203.41.20:8088\/pt1\/mysql:5.7/' ./dockerfile
cd ../
ls
vim copy.sh
进⾏后端的基础打包配置
mvn clean package -DskipTests
```
![](ruoyi-image/Pasted_image_20260529141836.png)
![](ruoyi-image/Pasted_image_20260529141847.png)
```
ls-la 查看一下
进入cd ruoyi-ui/ 配置前段基础配置
npm install
npm run build:prod
```
![](ruoyi-image/Pasted_image_20260529142047.png)
```
ll -t 看到那个dist
```
![](ruoyi-image/Pasted_image_20260529142105.png)

```
cd ../docker/
ls
执⾏脚本
sh copy.sh
```
![](ruoyi-image/Pasted_image_20260529142227.png)
```
ls
进⼊cd mysql/ 修改dockerfile
vim dockerfile
```
![](ruoyi-image/Pasted_image_20260529142244.png)
```
开启 ruoyi-mysql数据库
docker-compose up -d ruoyi-mysql
```
![](ruoyi-image/Pasted_image_20260529142301.png)
```
在 /mysql/⽬录下
进⼊容器并导⼊库
docker-compose cp ../sql/ry_config_20250224.sql ruoyi-mysql:/tmp
docker-compose cp ../sql/ry_20250523.sql ruoyi-mysql:/tmpdocker-compose exec -it ruoyi-mysql /bin/bash
mysql -u root -ppassword
show databases;
exit
cd ..
```
![](ruoyi-image/Pasted_image_20260529142326.png)
```
docker-compose exec -it ruoyi-mysql /bin/bash
mysql -u root -ppassword
会发现报错进行一下的更改
进入 cd ../docker/
docker-compose cp ../sql/ry_20250523.sql ruoyi-mysql:/tmp/
docker-compose cp ../sql/ry_config_20250224.sql ruoyi-mysql:/tmp/
docker-compose exec -it ruoyi-mysql /bin/bash
mysql -u root -ppassword
CREATE DATABASE ry-config DEFAULT CHARACTER SET utf8mb4 COLLATE
utf8mb4_general_ci; 创建数据库
```
![](ruoyi-image/Pasted_image_20260529142409.png)
show databases 查看数据库
![](ruoyi-image/Pasted_image_20260529142419.png)
```
source /tmp/ry_config_20250902``.sql; # 导入nacos所依赖的数据库
source /tmp/ry_20250523``.sql; # 导入system需要的依赖库
```
![](ruoyi-image/Pasted_image_20260529142513.png)
查看是否导⼊
![](ruoyi-image/Pasted_image_20260529142533.png)
#### 9 .部署redis 进⼊到cd /redis/

```
修改redis dockerfile

sed -i 's/FROM redis/FROM 10.203.41.20:8088\/pt1\/redis:7.0.4/' ./dockerfile 启动

redis docker-compose up -d ruoyi-redis 
```
#### 10 . 部署

```
nacos 修改nacos dockerfile sed -i 's/FROM nacos/nacos-server/FROM

10.203.41.20:8088/pt1/nacos/nacos-server:v2.3.2/' ./dockerfile 配置

HOSTS echo "192.168.10.162 ruoyi-nacos ruoyi-mysql ruoyi-redis" >> /etc/hosts # IP地址

改成⾃⼰本机IP 启动nacos docker-compose up -d ruoyi-nacos`
```
![](ruoyi-image/Pasted_image_20260529142638.png)
#### 11 .访问nacos并修改相关yml⽂件

主要修改，关于mysql和redis相关ip，修改前为127.0.0.1 → ruoyi-redis/ruoyi-mysql点击发布，确认发布
![](ruoyi-image/Pasted_image_20260529142656.png)
![](ruoyi-image/Pasted_image_20260529142708.png)
#### 12 .部署nginx cd/nginx/

```
修改nginx dockerfile

sed -i 's/FROM nginx/FROM 10.203.41.20:8088\/pt1\/nginx/' ./dockerfile 修改nginxconf，进入cd conf/ 将ruoyi-gateway替换成本地IP sed -i 's/ruoyi-gateway/10.203.41.33/' nginx /conf/nginx .conf`
```
![](ruoyi-image/Pasted_image_20260529142858.png)
![](ruoyi-image/Pasted_image_20260529142908.png)
#### 13 . 部署nginx服务并启动

```
cd docker
docker-compose up -d ruoyi-nginx
修改所有bootstrap.yml
该⽂件中指定了调⽤nacos的配置，需要将127.0.0.1修改为 ruoyi-nacos主机名称cd RuoYi-Cloud-v3.6.6
find . -name bootstrap.yml

```
使用find命令找出ruoyi根目录下所有的bootstrap.yml

find . -name bootstrap.yml

只需要修改src⽬录的源码⽂件下的bootstrap.yml

```
./ruoyi-modules/ruoyi-system/src/main/resources/bootstrap.yml
./ruoyi-modules/ruoyi-job/src/main/resources/bootstrap.yml
./ruoyi-modules/ruoyi-gen/src/main/resources/bootstrap.yml
./ruoyi-modules/ruoyi-file/src/main/resources/bootstrap.yml
./ruoyi-visual/ruoyi-monitor/src/main/resources/bootstrap.yml
./ruoyi-auth/src/main/resources/bootstrap.yml
./ruoyi-gateway/src/main/resources/bootstrap.yml
```

```
cd ruoyi-auth/src/main/resources
vim bootstrap.yml
cd ..
cd../../
cd ..
cd ruoyi-modules/ruoyi-gen/src/main/resources/
vim bootstrap.yml
cd ../../../../../
cd ruoyi-modules/ruoyi-file/src/main/resources/
vim bootstrap.yml
cd ../../../../../ruoyi-modules/ruoyi-system/src/main/resources
vim bootstrap.yml
cd ../../../../../ruoyi-modules/ruoyi-job/src/main/resources
vim bootstrap.yml
cd ../../../../../ruoyi-gateway/src/main/resources
vim bootstrap.yml
cd ../../../../../ruoyi-visual/ruoyi-monitor/src/main/resources
cd ../../../../ruoyi-visual/ruoyi-monitor/src/main/resources
```
![](ruoyi-image/Pasted_image_20260529143213.png)
#### 14 .验证
```
find . -name *.jar
```
![](ruoyi-image/Pasted_image_20260529143237.png)
#### 15 .启动jar服务
```
cd docker # 进入docker目录

sh copy.sh # 主要目的是该脚本中包含了把ruoyi后端所有的jar包copy到 docker/ruoyi不同的模块
```
![](ruoyi-image/Pasted_image_20260529143304.png)
```
cat > run.sh << 'EOF'

#!/bin/bash
nohup java -jar .``/ruoyi/auth/jar/ruoyi-auth``.jar > ruoyi-auth.log 2>&1 &
nohup java -jar .``/ruoyi/modules/job/jar/ruoyi-modules-job``.jar > ruoyi-modules-job.log 2>&1 &
nohup java -jar .``/ruoyi/modules/gen/jar/ruoyi-modules-gen``.jar > ruoyi-modules-gen.log 2>&1 &
nohup java -jar .``/ruoyi/modules/file/jar/ruoyi-modules-file``.jar > ruoyi-modules-``file``.log 2>&1 &
nohup java -jar .``/ruoyi/modules/system/jar/ruoyi-modules-system``.jar > ruoyi-modules-system.log 2>&1 &
nohup java -jar .``/ruoyi/visual/monitor/jar/ruoyi-visual-monitor``.jar > ruoyi-visual-monitor.log 2>&1 &
nohup java -jar .``/ruoyi/gateway/jar/ruoyi-gateway``.jar > ruoyi-gateway.log 2>&1 &
echo "所有服务启动中..."
EOF
sh run.sh
```
![](ruoyi-image/Pasted_image_20260529143429.png)
#### 16 . 查看服务启动
```
ps -ef|grep ruoyi- |grep -v grep # 查看进程，⼀共7个
```

![](ruoyi-image/Pasted_image_20260529143455.png)
再次进⼊到浏览器访问192.168.10.162:8848/nacos
![](ruoyi-image/Pasted_image_20260529143509.png)![](ruoyi-image/Pasted_image_20260529143517.png)

打开⼀个新的标签访问http://192.168.10.162
![](ruoyi-image/Pasted_image_20260529143549.png)