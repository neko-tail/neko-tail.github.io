---
tags: systemd linux
categories: linux
---

需求：写一个service，用来简化启动和杀死jar包的动作

<!--more-->

实现方式如下

```sh
# 编写服务脚本，/usr/lib/systemd/system/服务名.service
vim /usr/lib/systemd/system/test.service
# 加载
systemctl daemon-reload
# 设置开机自启，使用disable取消开机自启
systemctl enable test
# 使用方法注意服务名的位置不同，两者都可
systemctl {start|stop|restart|reload|status|enable|disable} test
service test {start|stop|restart|reload|status|enable|disable}
```

脚本内容：

```sh
# 用来定义服务的描述、文档、启动顺序和依赖关系等信息
[Unit]
# 给出当前服务的简单描述
Description=test.jar service
# 表示当前服务在哪些服务或目标之后启动，这里代表在网络服务启动后启动
After=network.target

# 用来定义服务的类型、用户、工作目录、执行命令等信息
[Service]
# 设置进程的启动类型
# simple代表将ExecStart进程作为服务的主进程
# 并且认为在创建完主服务进程之后，服务就启动完成
Type=simple
# 设置运行该服务的用户和组
User=root
Group=root
# 设置服务的工作目录
WorkingDirectory=/root/blog
# 设置启动服务时执行的命令
ExecStart=/usr/local/java/jdk-17.0.7/bin/java -jar /root/blog/blog.jar
# 设置关闭服务时执行的命令
# 这里是直接kill掉
ExecStop=/bin/kill -s TERM $MAINPID
# 设置服务在失败后的重启方式
# on-failure代表服务意外退出时自动重启服务
Restart=on-failure

# 用来定义如何安装或卸载该服务
# 使用enable和disable命令时需要
[Install]
WantedBy=multi-user.target
```

