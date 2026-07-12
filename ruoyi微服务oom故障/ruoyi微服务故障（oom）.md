1.进入目录
```
cd /srv/app/tools
```
首先从仓库克隆一下仓库的地址
```
git clone http://git.baway.work:10080/fault/java/deadlock.git
```
ls 查看一下
![](Pasted_image_20260608170948.png)

2.进入 deadlock目录下面 
```
cd /deadlock
```
利用mvn打包
```
mvn clean install
```
![](Pasted_image_ 20260608171150.png)
![](Pasted_image_20260608171210.png)
退出这个目录
cd ../
3.进入下面这个目录
```
cd RuoYi-Cloud-v3.6.6/docker/
ls 查看一下
```
4.创建一个备份目录
```
mkdir -p /srv/backup/20260608/ruoyi-gateway
```
将当前目录下的ruoyi-gateway的jar包替换到上面那个备份目录中
```
cp ./ruoyi/gateway/jar/ruoyi-gateway.jar /srv/backup/20260608/ruoyi-gateway/   
```
(备份当前运行的 jar)
查看一下是否成功
```
ll /srv/backup/20260608/ruoyi-gateway/
```
![](Pasted_image_20260608171725.png)
```
cp ../../deadlock/target/ruoyi-gateway.jar ./ruoyi/gateway/jar/ruoyi-gateway.jar 
```
(用新编译的 jar 替换运行目录下的 jar做故障使用的)
若果出现这个 直接填 y
![](Pasted_image_20260608172505.png)
5.查看gateway的进程
```
ps -ef|grep gateway
ps -ef | grep -i ruoyi- |grep -v grep
```
如果没有现实的话重新执行一下启动脚本    sh run.sh
再次查看服务进程
```
ps -ef | grep -i ruoyi- |grep -v grep
```
![](Pasted_image_20260608172934.png)
```
查看一下中间件是否存活    docker-compose ps
```
![](Pasted_image_20260608173102.png)
```
如果有不存活的重启一下    docker-compose restart ruoyi-mysql
```
![](Pasted_image_20260608173157.png)
```
docker-compose restart ruoyi-nacos
```
若果7个服务进程还是没有起来就全部杀死
```
ps -ef|grep ruoyi- |grep -v grep | awk '{print $2}' | xargs kill -9
杀死后查看进程     ps -ef | grep -i ruoyi- |grep -v grep
执行启动脚本  sh run.sh
查看进程  ps -ef | grep -i ruoyi- |grep -v grep
```
执行这条命令  **模拟向本地运行的后端服务发送登录请求**，以验证用户名和密码是否正确，并获取认证凭证（如 Token 或 Session ID）
```
curl -s --connect-timeout 2 -X POST [http://127.0.0.1:8080/auth/login](http://127.0.0.1:8080/auth/login) \ -H "Content-Type: application/json" \ -d '{"username": "admin", "password": "admin123", "code": "", "uuid": ""}'
```
![](Pasted_image_20260608173820.png)
复制上面的token到下面这条命令中，将你的名字改成所复制的内容
```
ab -c 30 -n 500 -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VyX2tleSI6IjY3MjcwNjBmLWVmZjktNDhjYi1hMzNmLTcyMGY5NGQ0OTBkZSIsInVzZXJuYW1lIjoiYWRtaW4ifQ.XfqytwuInAW_LlprU99GoGqk3OsogeEHpbyLCpUbixyXYt60StFrU9HbHfGNZ2KVwSK5ZGZWD1JyBV4DWqmdjg" "http://127.0.0.1:8080/route/analysis/check?mode=oom"
```
![](Pasted_image_20260608174111.png)
```
查看进程   ps -ef | grep -i ruoyi- |grep -v grep
```
浏览器中nacos页面7个服务会变成6个
查看ruoyi-gateway.log 日志
```
tail -n 100 ruoyi-gateway.log 
```
![](Pasted_image_20260608174344.png)
![](Pasted_image_20260608174407.png)
```
杀掉gateway的那个服务  kill -9 31690
ps -ef | grep -i ruoyi- |grep -v grep
```
![](Pasted_image_20260608174551.png)
回滚
```
cp /srv/backup/20260608/ruoyi-gateway/ruoyi-gateway.jar ./ruoyi/gateway/jar/ruoyi-gateway.jar 
```
执行cat run.sh
```
#!/bin/bash
 
nohup java -jar ./ruoyi/auth/jar/ruoyi-auth.jar > ruoyi-auth.log 2>&1 &
nohup java -jar ./ruoyi/modules/job/jar/ruoyi-modules-job.jar > ruoyi-modules-job.log 2>&1 &
nohup java -jar ./ruoyi/modules/gen/jar/ruoyi-modules-gen.jar > ruoyi-modules-gen.log 2>&1 &
nohup java -jar ./ruoyi/modules/file/jar/ruoyi-modules-file.jar > ruoyi-modules-file.log 2>&1 &
nohup java -jar ./ruoyi/modules/system/jar/ruoyi-modules-system.jar > ruoyi-modules-system.log 2>&1 &
nohup java -jar ./ruoyi/visual/monitor/jar/ruoyi-visual-monitor.jar > ruoyi-visual-monitor.log 2>&1 &
nohup java -jar ./ruoyi/gateway/jar/ruoyi-gateway.jar > ruoyi-gateway.log 2>&1 &
 
echo "所有服务启动中..."

```
执行最后一条命令     
```
nohup java -jar ./ruoyi/gateway/jar/ruoyi-gateway.jar > ruoyi-gateway.log 2>&1 &
再次查看进程    ps -ef | grep -i ruoyi- |grep -v grep
jstack -F 35845
出现以下报错是内存不足

Error: -F option used
Cannot connect to core dump or remote debug server. Use jhsdb jstack instead
```




