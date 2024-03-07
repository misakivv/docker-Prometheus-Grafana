# docker-Prometheus-Grafana

#### 介绍
基于centos7.9系统使用docker安装部署Prometheus+Grafana方式实现对Linux系统主机监控管理

#### 软件架构

![image-20240307164912048](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240307170927870-1716258726.png)
#### 安装教程

###### 一、docker安装

1、安装docker
2、启动docker并设置开机自启
3、下载镜像包
4、创建prometheus挂载目录
5、创建prometheus配置文件
6、启动prometheus
7、查看端口状态
8、访问地址

###### 二、部署grafana

1、创建挂载数据目录
2、设置权限
3、启动grafana
4、查看端口状态
5、访问地址

###### 三、部署node-exporter

1、启动node-exporter
2、查看端口状态
3、访问网址

###### 四、添加监控节点

1、安装docker
2、启动docker并设置开机自启
3、被监控主机安装node-exporter
4、安装并启动镜像
5、修改prometheus配置文件
6、重启prometheus
7、测试

###### 五、添加监控模版

1、添加数据源
2、添加模版
3、下载需要的dashboard页面

4、上传JSON文件

#### 使用说明

> 部署环境
> 系统版本：CentOS Linux release 7.9.2009 (Core)
> docker版本：Docker version 1.13.1
> 关闭防火墙 systemctl stop firewalld.service
> 禁止防火墙开机自启 systemctl disable firewalld.service
> 关闭selinux
> sed -i ‘s/SELINUX=enforcing /SELINUX=disabled/g’ /etc/selinux/config
> 重启系统即可 reboot

> 部署主机
> 监控主机：192.168.112.30（Prometheus+Grafana）
> 被监控主机：192.168.112.20（node-exporter）

#### 参与贡献

1.  Fork 本仓库
2.  新建 Feat_xxx 分支
3.  提交代码
4.  新建 Pull Request


#### 特技

1.  使用 Readme\_XXX.md 来支持不同的语言，例如 Readme\_en.md, Readme\_zh.md
2.  Gitee 官方博客 [blog.gitee.com](https://blog.gitee.com)
3.  你可以 [https://gitee.com/explore](https://gitee.com/explore) 这个地址来了解 Gitee 上的优秀开源项目
4.  [GVP](https://gitee.com/gvp) 全称是 Gitee 最有价值开源项目，是综合评定出的优秀开源项目
5.  Gitee 官方提供的使用手册 [https://gitee.com/help](https://gitee.com/help)
6.  Gitee 封面人物是一档用来展示 Gitee 会员风采的栏目 [https://gitee.com/gitee-stars/](https://gitee.com/gitee-stars/)
