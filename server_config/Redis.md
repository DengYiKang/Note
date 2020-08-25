# Redis

## 安装与配置

### 安装编译与测试依赖

```shell
sudo apt-get update
sudo apt-get install build-essential tcl
```

### 编译与安装

```shell
make
make test	#测试
sudo make install
```

### 配置Redis

```shell
#编辑/redis/redis.conf，找到supervised关键字，因为ubuntu16使用的是systemd，因此将其改为systemd
. . .

# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised systemd

. . .

#找到dir关键字

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir /var/lib/redis

. . .

# 找到daemonize关键字，yes配置后台运行
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes
```

### 配置Redis服务

```shell
sudo vim /etc/systemd/system/redis.service
```

```sh
[Unit]
Description=Redis In-Memory Data Store
After=network.target
# After属性表示在启动服务前应先联网

[Service]
User=redis
Group=redis
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
ExecStop=/usr/local/bin/redis-cli shutdown
Restart=always
# Restart表示若从某次故障恢复过来后重启

[Install]
WantedBy=multi-user.target
```

### 添加Redis用户组

```shell
sudo adduser --system --group --no-create-home redis
sudo mkdir /var/lib/redis
sudo chown redis:redis /var/lib/redis	#给予Redis的redis用户文件夹所有权
sudo chmod 770 /var/lib/redis	#其他用户无所有权
```

### 开启以及测试服务

#### 开启

```shell
sudo systemctl start redis
sudo systemctl status redis
```

#### 测试

```shell
redis-cli
127.0.0.1:6379> ping

Output
PONG
```

```shell
127.0.0.1:6379> set test "It's working!"

Output
OK
```

```shell
127.0.0.1:6379> get test

Output
"It's working!"
```

```shell
127.0.0.1:6379> exit
sudo systemctl restart redis
redis-cli
127.0.0.1:6379> get test

Output
"It's working!"
```

### 开机时启动

```shell
sudo systemctl enable redis

Output
Created symlink from /etc/systemd/system/multi-user.target.wants/redis.service to /etc/systemd/system/redis.service.
```

