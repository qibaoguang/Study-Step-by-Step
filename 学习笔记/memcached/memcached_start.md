[Memcached初级教程](http://memcached.org/)
===========

### 安装与启动
#### 1. install
<pre>
Debian/Ubuntu: apt-get install libevent-dev Redhat/Centos: yum install libevent-devel
wget http://memcached.org/latest
tar -zxvf memcached-1.x.x.tar.gz
cd memcached-1.x.x
./configure && make && make test && sudo make install
</pre>
#### 2. start
<pre>
/usr/local/bin/memcached -d -m 200 -u root -l 192.168.1.91 -p 12301 -c 1000 -P /tmp/memcached.pid
</pre>
相关解释如下：
<pre>
-d选项是启动一个守护进程，
-m是分配给Memcache使用的内存数量，单位是MB，这里是200MB
-u是运行Memcache的用户，如果当前为 root 的话，需要使用此参数指定用户。
-l是监听的服务器IP地址，如果有多个地址的话，我这里指定了服务器的IP地址192.168.1.91
-p是设置Memcache监听的端口，我这里设置了12301，最好是1024以上的端口
-c选项是最大运行的并发连接数，默认是1024，这里设置了256
-P是设置保存Memcache的pid文件，我这里是保存在 /tmp/memcached.pid
停止Memcache进程：
# kill `cat /tmp/memcached.pid`
也可以启动多个守护进程，但是端口不能重复.
</pre>
一开始说的“-d”参数需要进行进一步的解释:
<pre>
-d install 安装memcached
-d uninstall 卸载memcached
-d start 启动memcached服务
-d restart 重启memcached服务
-d stop 停止memcached服务
-d shutdown 停止memcached服务
</pre>
#### 3. 检查服务
3.1、查看启动的memcache服务：
<pre>
netstat -lp | grep memcached
</pre>
3.2、查看memcache的进程号（根据进程号，可以结束memcache服务：“kill -9 进程号”）
<pre>
ps -ef | grep memcached 
</pre>
