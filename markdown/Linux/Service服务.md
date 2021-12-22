### service服务注册、开机自启 

service 文件存放位置：/etc/systemd/system/

示例文件

```shell

[Unit]
Description=xiaoji
After=syslog.target
#After=network.target
#After=nezha-dashboard.service

[Service]
# Modify these two values and uncomment them if you have
# repos with lots of files and get an HTTP error 500 because
# of that
###
#LimitMEMLOCK=infinity
#LimitNOFILE=65535
Type=simple
User=root
Group=root
WorkingDirectory=/home/root/
ExecStart=/home/root/client xj.top1.pub:1200
Restart=always
#Environment=DEBUG=true

# Some distributions may not support these hardening directives. If you cannot start the service due
# to an unknown option, comment out the ones not supported by your version of systemd.
#ProtectSystem=full
#PrivateDevices=yes
#PrivateTmp=yes
#NoNewPrivileges=true
[Install]
WantedBy=multi-user.target

```



修改配置后重新载入配置文件

systemctl daemon-reload

```shell
# 查看日志
journalctl -u service-name.service
```



参考链接：https://www.cnblogs.com/ggzhangxiaochao/p/15039617.html