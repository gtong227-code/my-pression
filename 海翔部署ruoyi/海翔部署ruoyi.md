# 一、准备环境：

1、虚拟机配置：4C8G(最低为2C4G)
安装docker（29.1.0），docker-compose（v2.26.1），部署jdk(21.0.4),apache-maven(3.9.6),nodejs(v12.14.0)

```
yum -y install docker-ce
```
```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.35.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose #下载docker-compose，到/usr/local/bin/下面

sudo chmod +x /usr/local/bin/docker-compose #授权，给予执行的权限
```

写入/etc/profile

![](Pasted_image_20260622103656.png|697)

写入后执行：

```
source /etc/profile
```

执行完后执行一下命令，验证是否成功：

```
docker --version
docker-compose --version
java --version
mvn -v
node -v
```

```
systmectl stop firewalld  #关闭防火墙
systemctl disable firewalld  #永久关闭防火墙
setenforce 0  #临时关闭SElinux（重启后失效）
```

2、上传RuoYi-Cloud-v3.6.6.tar.gz包在/srv/app/tools/，并解压

```
tar -zxf RuoYi-Cloud-v3.6.6.tar.gz
```

3、解压完后，进入Ruoyi-Cloud-v3.6.6里面的docker目录

```
cd /RuoYi-Cloud-v3.6.6/docker
```

# 二、部署中间件

1、修改docker-compose.yml文件

1）将里面的四个镜像文件修改为镜像仓库的镜像

image: 10.203.41.20:8088/pt1/nacos/nacos-server:v2.3.2
image: 10.203.41.20:8088/pt1/mysql:5.7
image: 10.203.41.20:8088/pt1/redis:7.0.4
image: 10.203.41.20:8088/pt1/nginx

2）修改nginx相关的（找到ruoyi-nginx）

![](Pasted_image_20260622103731.png)
注释掉，保存退出

2、部署mysql

将数据库的sql文件复制到docker目录下，执行copy.sh脚本

```
sh copy.sh #注意；里面的sql的文件名称要修改为RuoYi-Cloud-v3.6.6/sql下的文件名称，要对应好
```
```
cd /srv/app/tools/RuoYi-Cloud-v3.6.6/docker/mysql
```

修改dockerfile文件（这个文件里的mysql镜像文件不是仓库里的，需要修改）
可使用sed替换命令，也可以直接进入文件里面修改，任选其一

```
sed -i 's/FROM mysql:5.7/FROM 10.203.41.20:8088\/pt1\/mysql:5.7/' ./dockerfile
```

修改完成后启动mysql

```
docker-compose up -d ruoyi-mysql
```

进入容器导入数据库

```
docker-compose cp sql/ry_20250523.sql ruoyi-mysql:/tmp/
docker-compose cp sql/ry_config_20250224.sql ruoyi-mysql:/tmp/

#进入mysql容器
docker-compose exec -it ruoyi-mysql /bin/bash
#登录数据库（用户：root  密码：password）
mysql -u root -ppassword
CREATE DATABASE  `ry-config` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
source /tmp/ry_config_20250224.sql;
#show databases; 查看全部数据库
use ry-cloud;  #进入ry-cloud库里面
source /tmp/ry_20250523.sql;  #要进入进入ry-cloud库里面执行，否则不会生效，数据未导入
#可以验证一下，执行：show tables;
```
3、部署Redis

进入目录：

```
cd /srv/app/tools/RuoYi-Cloud-v3.6.6/docker/redis
```

修改dockerfile（这个文件里的redis镜像文件不是仓库里的，需要修改）
可使用sed替换命令，也可以直接进入文件里面修改，任选其一

```
sed -i 's/FROM redis/FROM 10.203.41.20:8088\/pt1\/redis:7.0.4/' ./dockerfile

#修改完后启动redis
docker-compose up -d ruoyi-redis
```

4、部署nacos

1)切换目录

```
cd /srv/app/tools/RuoYi-Cloud-v3.6.6/docker/nacos
```

2)修改dockerfile（这个文件里的redis镜像文件不是仓库里的，需要修改）
可使用sed替换命令，也可以直接进入文件里面修改，任选其一

```
sed -i 's/FROM nacos\/nacos-server/FROM 10.203.41.20:8088\/pt1\/nacos\/nacos-server:v2.3.2/' ./dockerfile
```

3)配置hosts

```
echo "10.204.41.xxx  ruoyi-nacos ruoyi-mysql ruoyi-redis" >> /etc/hosts  #ip地址写自己的
```

4)启动nacos

```
docker-compose up -d ruoyi-nacos
```

5)打开浏览器访问nacos，192.168.23.232:8848/nacos

![](Pasted_image_20260622103751.png)

修改ruoyi-gateway-dev.yml配置文件（将host: localhost 改为 host: ruoyi-redis）

![](Pasted_image_20260622103807.png)

# 三、部署后端

1）修改配置文件
查找文件：
```
find . -name bootstrap.yml  # 要在/srv/app/tools/RuoYi-Cloud-v3.6.6目录下查找
```

修改路径上带有/src目录的文件

![](Pasted_image_20260622103831.png)

改为：（只改服务注册地址和配置中心地址，下图是我已经改过的，原本应该都是127.0.0.1:8080）

![](Pasted_image_20260622103851.png)

2）maven打包
在当前目录下打包（也就是/srv/app/tools/RuoYi-Cloud-v3.6.6）

```
mvn clean install
```

验证一下是否打包成功：

```
find . -name *.jar  #查看当前目录下所有jar包
```

![[Pasted image 20260622103907.png)

3）启动微服务

```
cd docker
sh copy.sh

cat > run.sh << 'EOF'
> #!/bin/bash
>  
> nohup java -jar ./ruoyi/auth/jar/ruoyi-auth.jar > ruoyi-auth.log 2>&1 &
> nohup java -jar ./ruoyi/modules/job/jar/ruoyi-modules-job.jar > ruoyi-modules-job.log 2>&1 &
> nohup java -jar ./ruoyi/modules/gen/jar/ruoyi-modules-gen.jar > ruoyi-modules-gen.log 2>&1 &
> nohup java -jar ./ruoyi/modules/file/jar/ruoyi-modules-file.jar > ruoyi-modules-file.log 2>&1 &
> nohup java -jar ./ruoyi/modules/system/jar/ruoyi-modules-system.jar > ruoyi-modules-system.log 2>&1 &
> nohup java -jar ./ruoyi/visual/monitor/jar/ruoyi-visual-monitor.jar > ruoyi-visual-monitor.log 2>&1 &
> nohup java -jar ./ruoyi/gateway/jar/ruoyi-gateway.jar > ruoyi-gateway.log 2>&1 &
>  
> echo "所有服务启动中..."
> EOF

sh run.sh
```

4)查看进程：

```
ps -ef|grep ruoyi- |grep -v grep

#杀死进程
ps -ef|grep ruoyi- |grep -v grep | awk '{print $2}' | xargs kill -9
```

# 四、部署前端

1、打包文件

进入目录：

```
cd /srv/app/tools/RuoYi-Cloud-v3.6.6/ruoyi-ui
```

打包：

```
npm run build:prod
```

打完包会生成一个名为dist的文件

![](Pasted_image_20260622103926.png)

2、部署nginx

1）切换目录：

```
cd /srv/app/tools/RuoYi-Cloud-v3.6.6/docker/nginx/
```

2）修改dockerfile（这个文件里的redis镜像文件不是仓库里的，需要修改）
可使用sed替换命令，也可以直接进入文件里面修改，任选其一

```
sed -i 's/FROM nginx/FROM 10.203.41.20:8088\/pt1\/nginx/' ./dockerfile
```

3）修改nginx.conf
与上面一样，可替换，也可修改

```
sed -i 's/ruoyi-gateway/192.168.23.232/' ./conf/nginx.conf  #ip改为自己的ip地址
```

4）把ruoyi-ui生成的dist目录copy到nginx目录下

```
cd /srv/app/tools/RuoYi-Cloud-v3.6.6/docker/

sh copy.sh
```

5）启动nginx

```
docker-compose  up -d ruoyi-nginx
```

# 五、测试访问

http://192.168.23.232/

![](Pasted_image_20260622104101.png)


# 六、常见问题

若依故障，常见问题（针对老师发的执行脚本）：

1、鉴权异常
![](Pasted_image_20260622104306.png)


原因：获取token失败或者token失效（在执行完脚本后，获取到的token失效了，并且没有替换成功，导致gateway报鉴权异常）

解决方法：

```
curl -s --connect-timeout 2 -X POST http://127.0.0.1:8080/auth/login   -H "Content-Type: application/json"   -d '{"username": "admin", "password": "admin123", "code": "", "uuid": ""}'  #手动获取token，查看能否获取到token
```

![](Pasted_image_20260622104323.png)

```
curl  -XGET -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VyX2tleSI6IjU0YTFjNjMxLTg1MTYtNGFhMC05YjE2LTI5OTI3ZDIzYWI3NyIsInVzZXJuYW1lIjoiYWRtaW4ifQ.cp6NWbvCAM-gr46G8m4MkWabWvkuKo4RyZt5OoEKicKs02g1X8-0HuXty_iE9uH2YfZk4LX_kHDj85QJ3Pidew" "http://127.0.0.1:8080/system/user/getInfo"  #拿到获取到的token，尝试登录，是否登录成功
```

修改/usr/local/bin/sys-telemetry脚本，将里面的token手动替换成自己获得token，然后执行

![](Pasted_image_20260622104333.png)

```
sh /usr/local/bin/sys-telemetry -x
```

2、ruoyi-gateway日志如果报too many open file的错误（原因：超过了最大文件打开数。影响：导致访问服务时变得很慢很慢）
需要修改文件描述符限制

```
ulimit -n  #先查看当前的

ulimit -n 65535  #临时修改

sudo vim /etc/security/limits.conf  #永久修改
* soft nofile 65535 
* hard nofile 65535 
* soft nproc 65535 
* hard nproc 65535
  
#修改完成后，重启ruoyi-gateway,再次访问
```