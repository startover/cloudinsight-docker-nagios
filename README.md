# Cloudinsight Agent 与 Nagios 集成

## 原理介绍

同大多数 Nagios 的可视化系统一样，Cloudinsight Agent 依赖于 Nagios 输出的指标数据，也即需要启用 Nagios 的性能数据处理功能（在 nagios.cfg 文件中，设置process_performance_data=1即可）。

## Nagios 安装配置

由于 Nagios 安装配置较为繁琐，这里通过 Docker 容器的方式来解决这个问题。

然而，Cloudinsight Agent 需要读取 Nagios 的配置文件且依赖于 Nagios 输出的指标数据，所以我们需要将这部分文件挂载到系统主机上。

* 创建 Nagios 配置文件

```
$ sudo mkdir -p /usr/local/nagios/data
$ sudo mkdir -p /usr/local/nagios/etc
$ sudo wget -O /usr/local/nagios/etc/nagios.cfg https://raw.githubusercontent.com/startover/cloudinsight-docker-nagios/master/nagios.cfg
$ sudo vi /usr/local/nagios/etc/nagios.cfg
...
# 启用 Nagios 的性能数据处理功能
process_performance_data=1

host_perfdata_command=process-host-perfdata
service_perfdata_command=process-service-perfdata

host_perfdata_file=/usr/local/nagios/data/host-perfdata
service_perfdata_file=/usr/local/nagios/data/service-perfdata

host_perfdata_file_template=[HOSTPERFDATA]\t$TIMET$\t$HOSTNAME$\t$HOSTEXECUTIONTIME$\t$HOSTOUTPUT$\t$HOSTPERFDATA$
service_perfdata_file_template=[SERVICEPERFDATA]\t$TIMET$\t$HOSTNAME$\t$SERVICEDESC$\t$SERVICEEXECUTIONTIME$\t$SERVICELATENCY$\t$SERVICEOUTPUT$\t$SERVICEPERFDATA$
...
```

* 挂载文件权限设置

由于挂载文件默认是root用户访问权限，需将其赋予容器内的 nagios 用户，如下：

```
$ sudo chown -R 999:999 /usr/local/nagios
```

* 运行 Nagios 容器

```
$ docker run -d --name docker-nagios \
             -h docker-nagios \
             -p 25 -p 80 \
             -v /usr/local/nagios/etc/nagios.cfg:/usr/local/nagios/etc/nagios.cfg \
             -v /usr/local/nagios/data:/usr/local/nagios/data \
             -v /etc/localtime:/etc/localtime:ro \
             quantumobject/docker-nagios
```

## Cloudinsight Agent 集成

* [安装](http://cloud.oneapm.com/#/settings) Cloudinsight Agent

* 配置 Nagios 监控

```
$ cd /etc/cloudinsight-agent/conf.d
$ sudo cp nagios.yaml.example nagios.yaml
$ sudo vi nagios.yaml
init_config:
  # check_freq: 30

instances:
  - nagios_conf: /usr/local/nagios/etc/nagios.cfg
```

* 重启 Cloudinsight Agent

```
$ sudo /etc/init.d/cloudinsight-agent restart
```


关于 Nagios 的介绍和相关配置，请参考下面几篇文章：  
http://www.cnblogs.com/mchina/archive/2013/02/20/2883404.html  
http://www.unixmen.com/install-configure-nagios-4-centos-7/  
https://kura.io/2010/03/21/configuring-nagios-to-monitor-remote-load-disk-using-nrpe/
