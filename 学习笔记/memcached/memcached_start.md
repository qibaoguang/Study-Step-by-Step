[Memcached初级教程](http://memcached.org/)
===========
### 一. 安装与启动
#### 1. install
```shell
Debian/Ubuntu: apt-get install libevent-dev 
Redhat/Centos: yum install libevent-devel
wget http://memcached.org/latest
tar -zxvf memcached-1.x.x.tar.gz
cd memcached-1.x.x
./configure && make && make test && sudo make install
````
#### 2. start
```shell
/usr/local/bin/memcached -d -m 200 -u root -l 192.168.1.91 -p 12301 -c 1000 -P /tmp/memcached.pid
```
相关解释如下：
```shell
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
```
一开始说的“-d”参数需要进行进一步的解释:
```shell
-d install 安装memcached
-d uninstall 卸载memcached
-d start 启动memcached服务
-d restart 重启memcached服务
-d stop 停止memcached服务
-d shutdown 停止memcached服务
```
#### 3. 检查服务
3.1、查看启动的memcache服务：
```shell
netstat -lp | grep memcached
```
3.2、查看memcache的进程号（根据进程号，可以结束memcache服务：“kill -9 进程号”）
```shell
ps -ef | grep memcached 
```

### 二. Memcached Java客户端（spymemcached）与Spring整合
net.spy.memcached.spring.MemcachedClientFactoryBean在net.spy.memcached.MemcachedClient每次使用的时候创建MemcachedClient的新实例。
<pre>
    <code>
&lt;bean id="memcachedClient" class="net.spy.memcached.spring.MemcachedClientFactoryBean"&gt;  
    &lt;property name="servers" value="host1:11211,host2:11211,host3:11211"/&gt;  
    &lt;property name="protocol" value="BINARY"/&gt;  
    &lt;property name="transcoder"&gt;  
    &lt;bean class="net.spy.memcached.transcoders.SerializingTranscoder"&gt;  
        &lt;property name="compressionThreshold" value="1024"/&gt;  
    &lt;/bean&gt;  
    &lt;/property&gt;  
    &lt;property name="opTimeout" value="1000"/&gt;  
    &lt;property name="timeoutExceptionThreshold" value="1998"/&gt;  
    &lt;property name="hashAlg" value="KETAMA_HASH"/&gt;  
    &lt;property name="locatorType" value="CONSISTENT"/&gt;   
    &lt;property name="failureMode" value="Redistribute"/&gt;  
    &lt;property name="useNagleAlgorithm" value="false"/&gt;  
&lt;/bean&gt;
    </code>
</pre>
属性说明：
<pre>
Servers :一个字符串，包括由空格或逗号分隔的主机或IP地址与端口号
Daemon :设置IO线程的守护进程(默认为true)状态
FailureMode :设置故障模式(取消，重新分配，重试)，默认是重新分配
HashAlg :设置哈希算法(见net.spy.memcached.HashAlgorithm的值)
InitialObservers :设置初始连接的观察者(观察初始连接)
LocatorType :设置定位器类型(ARRAY_MOD,CONSISTENT),默认是ARRAY_MOD
MaxReconnectDelay :设置最大的连接延迟
OpFact :设置操作工厂
OpQueueFactory :设置操作队列工厂
OpTimeout :以毫秒为单位设置默认的操作超时时间
Protocol :指定要使用的协议(BINARY,TEXT),默认是TEXT
ReadBufferSize :设置读取的缓冲区大小
ReadOpQueueFactory :设置读队列工厂
ShouldOptimize :如果默认操作优化是不可取的，设置为false(默认为true)
Transcoder :设置默认的转码器(默认以net.spy.memcached.transcoders.SerializingTranscoder)
UseNagleAlgorithm :如果你想使用Nagle算法，设置为true
WriteOpQueueFactory :设置写队列工厂
AuthDescriptor :设置authDescriptor,在新的连接上使用身份验证
</pre>
