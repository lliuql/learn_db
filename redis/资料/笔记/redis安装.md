## Redis 安装

[官网下载](https://redis.io/download)

```bash
# 安装 将redis 源码包上传到/opt

# 安装 gcc等编译环境 
yum install -y gcc


#  编译
cd /opt/redis-6.2.5

make 

# 安装命令
# 指定安装目录
make install PREFIX=/usr/local/redis




[root@projectapp redis-6.2.5]# mkdir -p /etc/redis
[root@projectapp redis-6.2.5]# cp redis.conf /etc/redis/6379.conf
[root@projectapp redis-6.2.5]# vim /etc/redis.conf 
#后台启动设置daemonize no改成yes

#启动服务
[root@projectapp redis-6.2.5]# redis-server /etc/redis/6379.conf 

# 查看进程是否启动
[root@projectapp redis-6.2.5]# ps aux|grep redis
root     18698  0.0  0.1 162404  9932 ?        Ssl  10:05   0:00 redis-server 127.0.0.1:6379
root     18712  0.0  0.0 112708   980 pts/0    S+   10:05   0:00 grep --color=auto redis

# 可以通过redis-cli shutdown 来关闭服务

# 设置开机自启
cp /opt/redis-6.2.5/utils/redis_init_script /etc/init.d/redis
chkconfig redis on



```

