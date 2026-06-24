首先创建一些目录
```
mkdir -p /srv/{data,logs,sh/compose,app}
ll /srv/
cd /srv/sh/
ls
cd compose/
mkdir -p zentao     在这里创建zentao目录
ls
cd zentao/
ls
mkdir -p data/zentao/      
ls
```
使用docker拉去镜像创建容器
使用一个脚本（快速便捷）
```
vim run.sh                                        这里是拉取镜像创建容器，创建定义网段
```
![[Pasted image 20260522160644.png|482]]
```
docker info 查看docker的信息状态
```
![[Pasted image 20260522163333.png]]
查看自己的网段
```
docker network ls
```
![[Pasted image 20260522165112.png]]
docker network 的5种
![[Pasted image 20260522170500.png]]

如果出现一下错误
![[Pasted image 20260522165237.png]]
查看一下详细信息
![[Pasted image 20260522170801.png]]

需要创建一个自己定义的网段
```
docker network create --subnet=172.172.172.0/24 zentaonet
```
查看一下镜像容像是否拉取并创建成功
![[Pasted image 20260522170930.png]]

执行脚本，表示镜像拉取创建成功
![[Pasted image 20260522165412.png]]
在浏览器中访问自己的ip加上端口，部署禅道
![[Pasted image 20260522165451.png]]
![[Pasted image 20260522165504.png]]
![[Pasted image 20260522165518.png]]
![[Pasted image 20260522165531.png]]
![[Pasted image 20260522165554.png]]
![[Pasted image 20260522165612.png]]
![[Pasted image 20260522165712.png]]
![[Pasted image 20260522165731.png]]
![[Pasted image 20260522165751.png]]
![[Pasted image 20260522170317.png]]

![[屏幕截图 2026-05-22 172141.png]]
![[屏幕截图 2026-05-22 172303.png]]
![[屏幕截图 2026-05-22 172317.png]]
![[屏幕截图 2026-05-22 172433.png]]
![[屏幕截图 2026-05-22 172507.png]]





