shell监控脚本-准备工作 
============
1. 准备监控机 
>linux 系统,普通服务器即可,要求: 
安装ssh 客户端,如果要发送手机短信,还需准备短信猫并且安装 gnokii或者 gammu。

2. 编辑ssh_config配置文件 
>/etc/ssh/ssh_config 配置文件，设置"GSSAPIAuthentication no"。
被监控的linux 编辑 /etc/ssh/sshd_config ，添加 UseDNS no ，最后重启sshd。

3. 使用密匙登录linux主机 

4. 建立sh目录，用于存放shell 脚本 
>mkdir -p /root/sh/crontab/log
>sh目录存放shell脚本 
>crontab/log目录存放错误信息 

5. 准备配置文件 
>cat /root/sh/CONFIG
>MOBILES="13xxxxxxxxx 18xxxxxxxxx 13xxxxxxxxx" 
>MAILS="dongnan@ywwd.net user2@ywwd.net" 
>
>ESXI_HOSTS="192.168.57.91 192.168.57.93" 
>PHYSICAL_HOSTS="192.168.57.112 192.168.0.1 192.168.57.99" 
>LINUX_WEB_HOSTS="192.168.57.82 192.168.57.70 10.0.100.72 10.0.100.73 10.0.100.75 10.0.100.76 >10.0.100.77 10.0.100.78" 
>WIN_WEB_HOSTS="10.0.100.81 10.0.100.83" 
>DB_SLAVE_HOSTS="10.0.100.82" 
>ALLHOSTS="\$ESXI_HOSTS \$PHYSICAL_HOSTS \$LINUX_WEB_HOSTS \$WIN_WEB_HOSTS \$DB_SLAVE_HOSTS"

 注意:此配置文件用于定义全局变量，包括ip 地址，邮件地址，电话号码等等。

6. crontab任务计划
>crontab -l
>\#ping
>*/1 * * * * /root/sh/chk_ping.sh >> /root/sh/cron.log 2>&1
>\#df
>*/1 * * * * /root/sh/chk_df.sh >> /root/sh/cron.log 2>&1
>\#load
>*/1 * * * * /root/sh/chk_load.sh >> /root/sh/cron.log 2>&1
>\#mysql_replicate
>*/1 * * * * /root/sh/chk_mysql_replicate.sh >> /root/sh/cron.log 2>&1
>\#web
>*/1 * * * * /root/sh/chk_web.sh >> /root/sh/cron.log 2>&1

注意:脚本执行时间]需按照脚本实际功能来制定，例如 chk_df。 监控服务器磁盘空间，每小时执行一次就可以了； 
所有监控脚本在 rhel5/centos5 下测试正常，其它linux 系统请自行测试。 
