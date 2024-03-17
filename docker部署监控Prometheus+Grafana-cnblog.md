[toc]

## 一、Prometheus简介

![71be210a68f24749a56d3f7117c77fd5](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123709341-225871448.png)
Prometheus 是一个开源的系统监控和报警系统，在 2012 年由 SoundCloud 公司创建，并于 2015 年正式发布。2016 年，Prometheus 正式加入 CNCF (Cloud Native Computing Foundation)，成为继kubernetes之后第二个在CNCF托管的项目, 现已广泛用于在容器和微服务领域中得到了广泛的应用，当然不仅限于此Prometheus 本身基于Go语言开发的一套开源的系统监控报警框架和时序列数据库(TSDB)。

Prometheus 的监控功能很完善和全面，性能也足够支撑上万台规模的集群。

网站：[https://prometheus.io/](https://prometheus.io/)

github：https://github.com/prometheus

## 二、Prometheus基本原理

Prometheus的基本原理是通过HTTP协议周期性抓取被监控组件的状态，任意组件只要提供对应的HTTP接口就可以接入监控。不需要任何SDK或者其他的集成过程。这样做非常适合做虚拟化环境监控系统，比如VM、Docker、Kubernetes等。输出被监控组件信息的HTTP接口被叫做exporter 。目前互联网公司常用的组件大部分都有exporter可以直接使用，比如Varnish、Haproxy、Nginx、MySQL、Linux系统信息(包括磁盘、内存、CPU、网络等等)。
其大概的工作流程是：
1、Prometheus server 定期从配置好的 jobs 或者 exporters 中拉 metrics，或者接收来自 Pushgateway 发过来的 metrics，或者从其他的 Prometheus server 中拉 metrics。
2、Prometheus server 在本地存储收集到的 metrics，并运行已定义好的 alert.rules，记录新的时间序列或者向 Alertmanager 推送警报。
3、Alertmanager 根据配置文件，对接收到的警报进行处理，发出告警。
4、在Grafana图形界面中，可视化查看采集数据。

## 三、Prometheus架构图

![image-20240307164912048](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123708832-1944530944.png)

## 四、Prometheus特性

1、多维度数据模型。

2、灵活的查询语言。

3、不依赖分布式存储，单个服务器节点是自主的。

4、通过基于HTTP的pull方式采集时序数据。

5、可以通过中间网关进行时序列数据推送。

6、通过服务发现或者静态配置来发现目标服务对象。

7、支持多种多样的图表和界面展示，比如Grafana等。

## 五、Prometheus组件

- Prometheus server是Prometheus架构中的**核心组件**，基于go语言编写而成，无第三方依赖关系，可以独立部署在物理服务器上、云主机、Docker容器内。主要用于收集每个目标数据，并存储为时间序列数据，对外可提供数据查询支持和告警规则配置管理。
  - Prometheus服务器可以对监控目标进行静态配置管理或者动态配置管理，**它将监控采集到的数据按照时间序列存储在本地磁盘的时序数据库中**（当然也支持远程存储），自身对外提供了自定义的PromQL语言，可以对数据进行查询和分析
- Client Library是用于检测应用程序代码的客户端库。在监控服务之前，需要向客户端库代码添加检测实现Prometheus中metric的类型。
- Exporter（数据采集）用于**输出被监控组件信息的HTTP**接口统称为Exporter（导出器）。目前互联网公司常用的组件大部分都有Expoter供**直接使用**，比如Nginx、MySQL、linux系统信息等。
- Pushgateway是指用于支持短期临时或批量计划任务工作的汇聚节点。主要用于短期的job，此类存在的job时间较短，可能在Prometheus来pull之前就自动消失了。所以针对这类job，设计成可以直接向Pushgateway推送metric，这样Prometheus服务器端便可以定时去Pushgateway拉去metric
- Pushgateway是prometheus的一个组件，prometheus server默认是通过exporter主动获取数据（默认采取pull拉取数据），pushgateway则是通过被动方式推送数据到prometheus server，用户可以写一些自定义的监控脚本把需要监控的数据发送给pushgateway， 然后pushgateway再把数据发送给Prometheus server
  - 总结就是pushgateway是普罗米修斯的一个组件，是通过被动的方式将数据上传至普罗米修斯。这个可以解决不在一个网段的问题
- Alertmanager主要用于处理Prometheus服务器端发送的alerts信息，对其去除重数据、分组并路由到正确的接收方式，发出**告警**，支持丰富的告警方式。
- Service Discovery：**动态发现**待监控的target，从而完成监控配置的重要组件，在容器环境中尤为重要，该组件目前由Prometheus Server内建支持

## 六、Prometheus服务发现

由于 Prometheus 是通过 Pull 的方式主动获取监控数据，也就是每隔几秒钟去各个target采集一次metric。所以需要手工指定监控节点的列表，当监控的节点增多之后，每次增加节点都需要更改配置文件，尽管可以使用接口去热更新配置文件，但仍然非常麻烦，这个时候就需要通过服务发现（service discovery，SD）机制去解决。
Prometheus 支持多种服务发现机制，可以自动获取要收集的 targets，包含的服务发现机制包括：azure、consul、dns、ec2、openstack、file、gce、kubernetes、marathon、triton、zookeeper（nerve、serverset），配置方法可以参考手册的配置页面。可以说 SD 机制是非常丰富的，但目前由于开发资源有限，已经不再开发新的 SD 机制，只对基于文件的 SD 机制进行维护。针对我们现有的系统情况，我们选择了静态配置方式。

## 七、部署环境

> 系统版本：CentOS Linux release 7.9.2009 (Core)
> docker版本：Docker version 1.13.1
> 关闭防火墙 systemctl stop firewalld.service
> 禁止防火墙开机自启 systemctl disable firewalld.service
> 关闭selinux
> sed -i ‘s/SELINUX=enforcing /SELINUX=disabled/g’ /etc/selinux/config
> 重启系统即可 reboot

## 八、部署主机

> 监控主机：192.168.112.30（Prometheus+Grafana）
> 被监控主机：192.168.112.20（node-exporter）

## 九、部署Prometheus

### 1、安装docker

> 在监控主机：192.168.112.30（Prometheus+Grafana）上操作

```bash
[root@server ~]# yum install wget.x86_64 -y	#使用Yum包管理器在系统上安装wget工具（适用于x86_64架构）
[root@server ~]# rm -rf /etc/yum.repos.d/*	#删除 /etc/yum.repos.d/ 目录下的所有文件。这个目录存放了Yum用来获取和更新软件包的仓库定义文件。通过删除现有仓库配置，管理员可以确保接下来将只使用新添加的仓库源。
[root@server ~]# wget -O /etc/yum.repos.d/Centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo    #使用wget工具从阿里云镜像站下载CentOS 7的官方软件仓库配置文件，并将其保存为 /etc/yum.repos.d/Centos-7.repo。这样做的目的是更换默认的CentOS基础软件源为阿里云提供的国内镜像源，以提高软件包下载速度。
[root@server ~]# wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo	#下载适用于CentOS 7的EPEL扩展仓库配置文件，并将其保存到 /etc/yum.repos.d/epel-7.repo
[root@server ~]# wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#下载Docker CE的官方仓库配置文件，同样是从阿里云镜像站下载，这样可以加速Docker CE相关软件包的下载。
[root@docker-server ~]# yum install docker-ce -y	#使用配置好的yum软件源来安装Docker CE社区版。
```

### 2、启动docker并设置开机自启

```bash
systemctl start docker
systemctl enable docker
```

### 3、下载镜像包

```bash
docker pull prom/node-exporter
```

![image-20240307121715886](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123708446-919987735.png)

```bash
docker pull prom/prometheus
```

![image-20240307121823449](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123708113-1797415197.png)

```bash
docker pull grafana/grafana
```

![image-20240307121943527](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123707739-2022257659.png)

### 4、创建prometheus挂载目录

```bash
mkdir /opt/prometheus
```

### 5、创建prometheus配置文件

```yaml
vi /opt/prometheus/prometheus.yml
global:
  scrape_interval:    15s
  evaluation_interval: 15s

scrape_configs:
- job_name: prometheus
  static_configs:
  - targets: ['localhost:9090']
    labels:
      instance: centos7

- job_name: grafana
  static_configs:
  - targets: ['192.168.112.30:9100']
    labels:
      instance: centos7
```

> 注：这里的IP：192.168.112.30就是本地localhost本地IP，为了实验方便，将prometheus和grafana搭建在同一个服务器上了。

### 6、启动prometheus

```bash
docker run  -d \
-p 9090:9090 \
-v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml  \
prom/prometheus
```

![image-20240307124955015](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123707343-1208443245.png)

### 7、查看端口状态

```bash
netstat -antupl |grep 9090
```

![image-20240307125053379](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123706996-1214193506.png)

### 8、访问地址

> 192.168.112.30:9090

![image-20240307125310850](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123706657-2047597454.png)

## 十、部署grafana

### 1、创建挂载数据目录

```bash
mkdir /opt/grafana-storage
```

### 2、设置权限

```bash
chmod 777 -R /opt/grafana-storage
```

> 注：因为这个文件需要写入所以要给一定的权限，这里为了方便测试给777，具体权限要根据具体实际情况而定。

### 3、启动grafana

```bash
docker run -d \
-p 3000:3000 \
--name=grafana \
-v /opt/grafana-storage:/var/lib/grafana \
grafana/grafana
```

### 4、查看端口状态

```bash
netstat -antupl | grep 3000
```

![image-20240307125848294](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123706251-889317494.png)

### 5、访问地址

> 192.168.112.30:3000
>
> 注：默认账号密码都是admin

![image-20240307130112386](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123705701-795456350.png)

## 十一、部署node-exporter

### 1、启动node-exporter

```bash
docker run -d -p 9100:9100 \
-v "/proc:/host/proc:ro" \
-v "/sys:/host/sys:ro" \
-v "/:/rootfs:ro" \
--net="host" \
prom/node-exporter
```

### 2、查看端口状态

```bash
netstat -antupl | grep 9100
```

![image-20240307150521211](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123705208-935963798.png)

### 3、访问网址

> 192.168.112.30:9100
> ![image-20240307150357843](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123704882-57708271.png)

## 十二、添加监控节点

> 在被监控主机：192.168.112.20（node-exporter）上操作

### 1、安装docker

```bash
yum -y install docker
```

### 2、启动docker并设置开机自启

```bash
systemctl start docker
systemctl enable docker
```

### 3、被监控主机安装node-exporter

```bash
docker pull prom/node-exporter
```

![image-20240307150920565](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123704527-1228512104.png)

### 4、安装并启动镜像

```bash
docker run -d -p 9100:9100 \
-v "/proc:/host/proc:ro" \
-v "/sys:/host/sys:ro" \
-v "/:/rootfs:ro" \
--net="host" \
prom/node-exporter
```

### 5、修改prometheus配置文件

> 在监控主机：192.168.112.30（Prometheus+Grafana）上操作

```yaml
cat /opt/prometheus/prometheus.yml

global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: centos7
  - job_name: grafana
    static_configs:
      - targets: ['192.168.112.30:9100']
        labels:
          instance: centos7
  - job_name: harbor
    static_configs:
      - targets: ['192.168.112.20:9100']
        labels:
          instance: centos7
```

### 6、重启prometheus

```bash
docker restart 6decfdbc9fa4
```

![image-20240307151602332](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123704165-1317607421.png)

### 7、测试

- 网页登录prometheus
- 点击Status下拉选项，选择Targets。
  ![image-20240307151840533](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123703856-262543273.png)
- 如下图说明添加监控节点已完成
  ![image-20240307153900365](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123703454-1168708595.png)

## 十三、添加监控模版

### 1、添加数据源

- 网页登陆grafana

![image-20240307154102695](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123702882-949758752.png)

- 依次点击左侧`Connections` `Data source` `Add data source`

![image-20240307154501201](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123702476-330894838.png)

- 在`Add data source`中找到`Prometheus`

![image-20240307154704756](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123702090-323318087.png)

- 点击`Prometheus`，设置名字和IP地址

![image-20240307155257087](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123701647-1152399988.png)

---

![image-20240307155600095](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123701212-812719806.png)

> 注：如上图点击Save & Test，如果出现绿色的，说明ok了。

### 2、添加模版

- 点击左上角+号，点击`Import dashboard`

![image-20240307155821443](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123700891-2056515341.png)

---

![image-20240307155917633](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123700470-2071411370.png)

### 3、下载需要的dashboard页面

- > **Grafana官方为我们提供了很多dashboard页面，可直接下载使用。浏览器访问 https://grafana.com/grafana/dashboards下载所需要的dashboard页面**

![image-20240307160720919](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123700046-1426026387.png)

- **本示例使用的Dashboard如下图所示，链接如下**：

[[Node Exporter Full | Grafana Labs](https://grafana.com/grafana/dashboards/1860-node-exporter-full/)]([Node Exporter Full | Grafana Labs](https://grafana.com/grafana/dashboards/1860-node-exporter-full/))

- **下载JSON文件，用于导入。**

  ![image-20240307163651983](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123659579-931260346.png)

- #### 上传JSON文件

![image-20240307163906311](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123659200-1110944153.png)

### 4、效果演示

![image-20240307164002390](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123658802-2007707260.png)

---

![image-20240307164021865](https://img2023.cnblogs.com/blog/3332572/202403/3332572-20240317123658320-1487792475.png)