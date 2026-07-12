`上面是虚拟机故障机创建`
`cd /etc/yum.repos.d/`
    `2  ls`
    `3  mv CentOS.repo /tmp/`
    `4  rm -rf *`
    `5  mv /tmp/CentOS.repo .`
    `6  ls`
    `7  exit`
    `8  w`
    `9  top`
   `10  pkill -9 stress`
   `11  w`
   `12  top`
   `13  ll /var/spool/cron/`
   `14  cat /var/spool/cron/sysbackup` 
   `15  ll /tmp/.hidden_stress_script.sh` 
   `16  tail -fn 100 /var/log/messages` 
   `17  cat /etc/rc.local` 
   `18  cat /tmp/.create_stress_load.sh` 
   `19  cat /tmp/.hidden_stress_script.sh` 
   `20  vim /etc/rc.local` 
   `21  vi /etc/rc.local` 
   `22  ll /tmp/ -al`
   `23  w`
   `24  top`
   `25  cat /var/spool/cron/sysbackup` 
   `26  tail -f /tmp/sysbackup.log` 
   `27  \reboot`
   `28  ifcon`
   `29  ip a`
   `30  cd /etc/sysconfig/network-scripts/`
   `31  ls`
   `32  ifup ifcfg-ens192`
   `33  ip a`
   `34  ifup ifcfg-ens192`
   `35  ip a`
   `36  passwd root`
   `37  w`
   `38  top`
   `39  cat /etc/rc.local` 
   `40  ls`
   下面是解决单用户模式以及负载高的解决方法
   ![[屏幕截图 2026-05-25 141807.png]]
   这里开启虚拟机在开始界面按e进入修改
   ![[屏幕截图 2026-05-25 142436.png]]
   `找到以 linux16 开头的行（通常在末尾有 rhgb quiet）。`

- `将 ro（只读）改为 rw（读写）`
    
- `在行尾添加 init=/sysroot/bin/sh`  
    `修改后类似：`
    
1. `然后按 Ctrl+X 或 F10 启动。`
    
2. `**进入救援模式并重置密码**`  
    `系统会直接进入 shell 环境，执行：`
`text`
`touch /.autorelabel`
`linux16 /vmlinuz-... rw ... init=/sysroot/bin/sh`
   `如果不想等待自动 relabel，也可以运行 restorecon -v /etc/shadow。`
   `**重启系统**`
   `exit`
`reboot`

   `41  vi /etc/sysconfig/network-scripts/ifcfg-ens33`
   `42  systemctl restart network`
   `43  free -h`
   ![[屏幕截图 2026-05-25 143606.png]]
   

   `44  df -h`
   `45  top`
   ![[屏幕截图 2026-05-25 144602.png]]
   ![[屏幕截图 2026-05-25 144647.png]]![[屏幕截图 2026-05-25 144653.png]]
   ![[屏幕截图 2026-05-25 144703.png]]
   ![[屏幕截图 2026-05-25 144714.png]]
   ![[屏幕截图 2026-05-25 151619.png]]
   
   
   `46  ping www.baidu.com`
   `47  yum install -y vim`
   `48  ls`
   `49  vi /etc/sysconfig/network-scripts/ifcfg-ens33`
   `50  ps aux | grep stress`
   `51  pkill -9 stress`
   `52  uptime`
   `53  pkill -9 stress`
   `54  ps aux | grep stress`
   `55  uptime`
   `56  top -bn1 | head -10`
   `57  crontab -u sysbackup -l`
   `58  crontab -u sysbackup -r`
   `59  rm -f /tmp/.hidden_stress_script.sh`
   `60  rm -f /tmp/sysbackup.log`
   `61  uptime`
   `62  systemctl list-timers --all | grep -i stress`
   `63  top`
   `64  cat /etc/crontab`
   `65  ls -la /etc/cron.d/ /etc/cron.hourly/ /etc/cron.daily/ /etc/cron.weekly/ /etc/cron.monthly/`
   `66  grep -r "stress\|hidden_stress" /etc/cron* 2>/dev/null`
   `67  systemctl list-timers --all | grep -i stress`
   `68  cat /etc/rc.local`
   `69  grep -r "stress" /home/sysbackup/.bash* /root/.bash* 2>/dev/null`
   `70  cd /tmp/`
   `71  ll -a`
   `72  cd .hidden_stress_script.sh` 
   `73  vim .hidden_stress_script.sh` 
   `74  rm -rf .hidden_stress_script.sh` 
   `75  chmod  -i .hidden_stress_script.sh` 
   `76  chattr   -i .hidden_stress_script.sh` 
   `77  rm -rf .hidden_stress_script.sh` 
   `78  top`
   `79  pkill -9 | grep stress`
   `80  pkill -9 stress`
   `81  top`
   `82  ls`
   `83  cd ..`
   `84  cd` 
   `85  top`