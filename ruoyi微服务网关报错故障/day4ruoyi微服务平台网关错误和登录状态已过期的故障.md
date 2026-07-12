
**第一部分网关错误的创建以及解决办法**

1 .下载并执行远程脚本的一键初始化命令，常见于快速部署环境
curl -sSO [http://10.203.41.72:9000/init_env.sh](http://10.203.41.72:9000/init_env.sh)&& chmod +x init_env.sh && ./init_env.sh && rm -f init_env.sh
![](Pasted_image_20260529150953.png)
![](Pasted_image_20260529151107.png)
2 . 查看ruoyi-gateway（ruoyi网关日志）
tail -fn 10 ruoyi-gateway.log
![](Pasted_image_20260529151253.png)

出现一个问题 
14:58:24.337 [reactor-http-epoll-2] ERROR c.r.g.h.GatewayExceptionHandler - [handle,52] - [网关异常处理]请求路径:/system/user/getInfo,异常信息:io.netty.channel.unix.Errors$NativeIoException: newSocketStream(..) failed: Too many open files   (too many open files)
![](Pasted_image_20260529152616.png)
修改最大文件数系统限制
ulimit -n  插看当前最大的限制
![](Pasted_image_20260529153003.png)
ulimit -n 5000 （临时修改）
![](Pasted_image_20260529153228.png)
杀掉那个关于网关的进程
![](Pasted_image_20260529153705.png)

查看启动ruoyi7个插件的脚本
![](Pasted_image_20260529153334.png)
找到ruoyi-gateway的那一条并在后台执行
nohup java -jar ./ruoyi/gateway/jar/ruoyi-gateway.jar > ruoyi-gateway.log 2>&1 &
![](Pasted_image_20260529153728.png)
查看一下进程
ps -ef|grep ruoyi- |grep -v grep
![](Pasted_image_20260529153806.png)
检查一下服务版本
![](Pasted_image_20260529153841.png)
再次查看一下ruoyi-gateway的日志
![](Pasted_image_20260529153925.png)


**第二部分登录状态过期的解决办法**
下载并执行远程脚本的一键初始化命令，常见于快速部署环境
curl -sSO [http://10.203.41.72:9000/init_env.sh](http://10.203.41.72:9000/init_env.sh) && chmod +x init_env.sh && ./init_env.sh && rm -f init_env.sh
tail  -fn 100  ruoyi-gateway.log
![[img_v3_02125_021fd107-ea37-4ae3-a5a7-9485701baf1g.jpg]]
上面的是临时修改，如果想要永久修改有以下的方法
方法一：直接追加（推荐，最快） bash 复制 # 直接追加到文件末尾
cat >> /etc/security/limits.conf << 'EOF'

* soft nofile 65535
* hard nofile 65535
root soft nofile 65535
root hard nofile 65535
EOF

# 验证是否写入成功
tail -6 /etc/security/limits.conf
 方法二：用 vim 手动编辑 bash 复制 vim /etc/security/limits.conf
 进入 vim 后： 1. 按 G — 跳到文件最后一行 2. 按 o — 在当前行下方新建一行进入插入模式 3. 输入以下内容：  code 复制 * soft nofile 65535
* hard nofile 65535
root soft nofile 65535
root hard nofile 65535
 4. 按 Esc — 退出插入模式 5. 输入 :wq — 保存并退出  写入后还需要做 bash 复制 # 修改 systemd 限制（否则通过 systemd 启动的服务不生效）
mkdir -p /etc/systemd/system.conf.d/
cat > /etc/systemd/system.conf.d/limits.conf << 'EOF'
[Manager]
DefaultLimitNOFILE=65535
EOF

# 重载 systemd 配置
systemctl daemon-reload

# 验证新限制（重新登录或开新终端后）
ulimit -n




测试登录接口、获取 Token（身份令牌）的命令。
curl -s --connect-timeout 2 -X POST [http://127.0.0.1:8080/auth/login](http://127.0.0.1:8080/auth/login) -H "Content-Type: application/json" -d '{"username": "admin", "password": "admin123", "code": "", "uuid": ""}' #手动获取token，查看能否获取到token

#以下是注释
逐步拆解
bash

复制
curl -s --connect-timeout 2 \
 -X POST http://127.0.0.1:8080/auth/login \
 -H "Content-Type: application/json" \
 -d '{"username": "admin", "password": "admin123", "code": "", "uuid": ""}'
部分	含义
curl -s	静默发送请求，不显示进度信息
--connect-timeout 2	连接超时 2 秒
-X POST	使用 POST 方法
-H "Content-Type: application/json"	请求体是 JSON 格式
-d '{...}'	POST 请求的请求体数据
实际在做什么
模拟前端调用后端的登录接口：

请求地址：http://127.0.0.1:8080/auth/login
用户名：admin
密码：admin123
成功响应会返回类似这样的 JSON：

json

复制
{
 "code": 200,
 "msg": "操作成功",
 "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
拿到 token 之后，后续请求（比如获取用户信息 /system/user/getInfo）就需要在请求头里带上这个 Token 来证明身份。

你之前遇到的"无token"问题
你之前的请求 code: 50001（无 Token）就是因为没有先调用 /auth/login 获取 Token，直接调了需要认证的接口，所以被拒绝。

现在你改完 ulimit 后，可以先用这条命令验证登录是否正常，再重新测试 /system/user/getInfo

token 访问需要认证的
curl -XGET -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VyX2tleSI6IjU0YTFjNjMxLTg1MTYtNGFhMC05YjE2LTI5OTI3ZDIzYWI3NyIsInVzZXJuYW1lIjoiYWRtaW4ifQ.cp6NWbvCAM-gr46G8m4MkWabWvkuKo4RyZt5OoEKicKs02g1X8-0HuXty_iE9uH2YfZk4LX_kHDj85QJ3Pidew" "[http://127.0.0.1:8080/system/user/getInfo](http://127.0.0.1:8080/system/user/getInfo)" #拿到获取到的token，尝试登录，是否登录成功

注释解读逐步拆解 bash 复制 curl -XGET \
 -H "Authorization: Bearer eyJhbGci..." \
 "http://127.0.0.1:8080/system/user/getInfo"
 部分 含义   -XGET 发送 GET 请求  -H "Authorization: Bearer ..." 在请求头里带上 Token（身份凭证）  /system/user/getInfo 获取当前登录用户的信息接口   Bearer Token 是什么 Bearer Token 就是你上一条命令 /auth/login 成功返回的那个 token 字段的值。把它放在请求头的 Authorization: Bearer <token> 里，就相当于告诉服务器**“我是用 admin 账号登录的那个人”**。 和你之前报错的关系 你之前 code: 50001（无 Token）就是因为请求没带这个头，现在带上 Token 再请求： • 成功 → 返回用户信息（用户名、角色、权限等） • 失败 → 可能返回 401 未授权或其他错误  验证流程 你现在应该这样测试： bash 复制 # 第1步：登录获取token
curl -s -X POST http://127.0.0.1:8080/auth/login \
 -H "Content-Type: application/json" \
 -d '{"username":"admin","password":"admin123","code":"","uuid":""}'

# 第2步：用返回的token访问用户信息（把 <token> 替换成上面返回的值）
curl -XGET \
 -H "Authorization: Bearer <token>" \
 "http://127.0.0.1:8080/system/user/getInfo"


这就是 /auth/login 接口返回给你的 JWT Token（身份令牌）
eyJhbGciOiJIUzUxMiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VyX2tleSI6IjhkZjQ2ODk5LWQ1MmMtNGY3Yi04MDc1LTBkOTE4YzFmYWY0YSIsInVzZXJuYW1lIjoiYWRtaW4ifQ.__Rlg94oaSriviTZ9LSkYcUGl_pJG__l0SWwOnZpiWdwFbNCQ2KaQBHGuGLoBxAOLfRkHFgLzo74KMZNm-QLvA

注释解读以及该怎么执行
code 复制 eyJhbGciOiJIUzUxMiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VyX2tleSI6IjhkZjQ2ODk5LWQ1MmMtNGY3Yi04MDc1LTBkOTE4YzFmYWY0YSIsInVzZXJuYW1lIjoiYWRtaW4ifQ.__Rlg94oaSriviTZ9LSkYcUGl_pJG__l0SWwOnZpiWdwFbNCQ2KaQBHGuGLoBxAOLfRkHFgLzo74KMZNm-QLvA
 由三部分组成，用 . 分隔： 部分 含义   eyJhbGciOiJIUzUxMiJ9 Header（头部，声明算法是 HS512）  eyJ1c2VyX2lkIjox... Payload（载荷，里面存着用户信息）  __Rlg94oaSrivi... Signature（签名，用于防篡改）   解码中间的 Payload 部分，里面包含： json 复制 {
 "user_id": 1, ← 用户ID是1
 "user_key": "8df46...", ← 用户唯一标识
 "username": "admin" ← 用户名是admin
}
 简单理解 • 前半段（header.payload）是"身份证信息"，可以被 Base64 解码看到内容 • 后半段（signature）是"防伪印章"，服务器用密钥校验，没法伪造  现在你应该做的是 把这条命令里的 <token> 替换成你刚才拿到的这个字符串，然后执行： bash 复制 curl -XGET \
 -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VyX2tleSI6IjhkZjQ2ODk5LWQ1MmMtNGY3Yi04MDc1LTBkOTE4YzFmYWY0YSIsInVzZXJuYW1lIjoiYWRtaW4ifQ.__Rlg94oaSriviTZ9LSkYcUGl_pJG__l0SWwOnZpiWdwFbNCQ2KaQBHGuGLoBxAOLfRkHFgLzo74KMZNm-QLvA" \
 "http://127.0.0.1:8080/system/user/getInfo"
 返回用户信息就说明登录成功了 ✅

停止并审查系统 telemetry（遥测）服务的命令。

/etc/systemd/system/sys-telemetry.service
systemctl stop sys-telemetry.service
vim  /usr/local/bin/sys-telemetry
 sh  /usr/local/bin/sys-telemetry -x

注释解读
逐步拆解
bash

复制
# 第1步：查看这个 systemd 服务的配置文件
cat /etc/systemd/system/sys-telemetry.service
↑ 确认这个服务是什么、怎么配置的（里面会写明 ExecStart 指向哪个脚本）

bash

复制
# 第2步：停止这个服务（不再运行）
systemctl stop sys-telemetry.service
↑ 临时停止，停止当前正在运行的进程

bash

复制
# 第3步：用 vim 查看这个脚本的内容
vim /usr/local/bin/sys-telemetry
↑ 确认这个脚本具体在做什么（是否在偷偷收集/上报数据）

bash

复制
# 第4步：用 debug 模式重新运行这个脚本
sh /usr/local/bin/sys-telemetry -x
↑ -x 会打印出脚本执行的每一步（类似 trace），用来排查它是否有问题

简单总结
步骤	操作	目的
1	查看服务配置	了解这个服务是谁创建的
2	停止服务	阻止它继续运行
3	查看脚本内容	确认它在做什么
4	debug 运行	排查脚本行为

执行完登录状态过期之后再去执行网关错误的步骤