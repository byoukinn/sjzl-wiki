# linux重启Tomcat版的DTS与MongoDB

如果部署过DTS项目，就会知道DTS分`Window`版本和`Linux`版本。

其中`Linux`版本下又分成`微服务版本`和`tomcat版本`，微服务版本直接通过docker或者容器平台操作重启即可，tomcat版会有一些区别，此处说明正常关闭与启动服务操作
要求先启动MongoDB，后启动DTS，否则DTS重试连接过多会罢工不再尝试连接，需要再重启DTS才会继续尝试连接。

## 一、MongoDB重启

!> 不同人部署路径与名称可能不一致

### 1.查看MongoDB进程
先检查数据库进程是否存活（注意：进程ID不一定是下图这个）
```shell
> ps aux | grep mongo
root	59086	0.1	0.9	10293410	151136	??	S	2:48下午	0:01.11	mongod --config /etc/mongod.conf
root	59097	0.0	0.0	453612109	1344	s000	S+	2:48下午	0:00.00	grep mongo
```

### 2. 尝试退出MongoDB
通过以下方式的关闭正在运作的mongo，下次启动可能会报错，需要手动重启mongo进程

1. 在mongo会话中直接关闭Terminal终端 或者 cmd
2. 直接执行kill结束MongoDB进程
3. 在mongo会话中 ctrl + D
4. 在mongo会话中 ctrl + C
5. 非正常重启

所以要掌握正常退出的方式

```shell
## 正常退出方法：
> mongod --shutdown --dbpath=/home/mongodb/data

## 如果退出不了则强制结束进程：
> kill -9 59086 # 59086是MongoDB进程号
```


### 3.找到MongoDB的安装路径

如果直接输入mongod没有报错：`该命令不存在`，代表环境变量配置好了，跳过这步

```shell
> find / -name mongod
/usr/local/bin/mongod
```

### 4.找到MongoDB的数据目录和日志目录

```shell
> find / -name collection-0-* # collection-N-* 是mongo的数据文件前缀
/home/data/collection-0-1198590742929274831.wt
/home/data/collection-0--4010924731385948638.wt
/home/data/collection-0-4743711213823132714.wt
/home/data/collection-0-7820424835517045838.wt

> find / -name mongodb.log
/log/mongodb.log
```

### 5.开启MongoDB服务

```shell
# mongod没有配置系统环境时，请输全路径
# --dbpath 填写MongoDB存放数据的目录（一般目录名带data） 
# --logpath 填写日志文件名（一般日志文件名叫mongodb.log）
# --auth 开启密码校验模式，必须输入账号密码才能进入数据库查询
# --fork 将本命令挂载至后台子进程
> mongod --dbpath=/home/mongodb/data --bind_ip=0.0.0.0 --auth --fork --logpath=/log/mongodb.log 
```

### 6.测试MongoDB是否正常启动

```shell
# 方法一
> mongo -u admin -p pass --authenticationDatabase=admin -d=Restcloud_DTS
mongo#>
# 显示没有报错则登陆数据库成功，登录成功代表正常启动
# -u 后是账号
# -p 后是密码
# -d 后是所用数据库名，可以在【DTS系统设置】-【关于平台】-【平台授权信息】-【MongoDB数据库】看到
# --authenticationDatabase 是校验数据库，通常写admin即可
# 账号密码忘记了去看"dts部署位置\tomcat\webapps\ROOT\WEB-INF\class\application.properties"这个文件

# 方法二
> curl –l http://127.0.0.1:27017
It looks like you access MongoDB over HTTP on the native driver port.
# 如果curl显示上面内容，则代表启动成功
```



## 二、DTS重启

### 1. 查看dts是否正常

通过登录该服务器的8080端口网页，如果能正常访问，则代表dts正常，显示无法访问代表DTS服务器挂了

### 2. 关闭dts服务器
```shell
> ps -ef | grep java 
root	24515	0.1	0.9	10293410	151136	??	S	2:48下午	0:01.11	java -Dcatalina.home=/home/tomcat org.apache.catalina.startup.Bootstrap start
root	51634	0.0	0.0	453612109	1344	s000	S+	2:48下午	0:00.00	grep java

> kill -9 24515 # 结束tomcat进程
```

### 3. 启动DTS所属tomcat

```shell
> cd /home/tomcat/bin # 你的DTS所在TOMCAT安装目录
> ./startup.sh
```

### 4. 验证启动成功

查看日志

```bash
> cd /tomcat安装目录/logs
> tail -100f ./catalina.out
...
...
... done in 58.4122 second！
...
> ^C
# 显示上面那句话就代表启动成功，一般需要一分钟到三分钟
```

访问dts平台网页查看是否可登录即可。

