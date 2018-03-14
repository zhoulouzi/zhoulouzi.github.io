---
title: "prometheus base"
date: 2017-04-05T21:48:15+08:00
draft: false
slug: "prometheus"
tags:
- prometheus
- influxDB
archives:
- docker-monitor
categories:
- docker
- prometheus
clearReading: true
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355321/Real_gaggav.png
thumbnailImagePosition: right
autoThumbnailImage: yes
metaAlignment: center
comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
---

>prometheus是SoundCloud开源的监控警报系统:
1. 多维度的数据模型(time series identified by metric name and key/value pairs)
2. promQL强大灵活的查询语言
3. 不依赖分布式存储
4. HTTP pull模式 收集 time series 这点跟influxdb的push模式不一样。
5. 通过pushgateway来支持push time series，适合short-lived job。
6. 可以通过服务发现或者静态配置scarpe目标
7. 支持多种模式的图形和仪表盘


## DATA MODEL
prometheus基本上可以利用time series 存储所有的数据：streams of timestamped values belonging to the same metric and the same set of labeled dimensions.
除此之外，还可以通过query生成临时衍生的time series。
Metric name and labels

每一个 time series 都是 被一个 metric name 和 一组 key-value 集合（labels） 唯一定义的。
metric name 代表一个系统被监测的功能。 eg：http_requests_total  命名规则是 符合 [a-zA-Z_:][a-zA-Z0-9_:]* 这个正则表达式。

labels 可以使prometheus 的数据多维度。对于相同metric name 给定了 labels组合 都是一个 特定维度 的 一个metric的实例。
query languages 允许 过滤和聚合通过这些维度，更改任何一个label都将新建一个新的time series

给定一组 metric name 和label
> \<metric name\>{\<label name\>=\<label value\>, ...}
> api_http_requests_total{method="POST", handler="/messages"}

metric types:
    prometheus 的 客户端库提供了4个核心的 metric type，不过只是在客户端库里存在，server端存的还是 untyped time series.
1. counter
    计数器，很好理解，代表一个只会增长的值。 用例：已经完成的请求数，任务完成数
2. gauge
    比计数器多个 减少。用例：内存使用值，the number of running goroutines.
3. histogram
    暂未理解
4. summary
    暂未理解，官方文档的最佳实践里有举例，学完再回来改

## Jobs and instances
对prometheus 来说每个scraped target都是一个实例。他们的集合可以称为job
这个在配置prometheus的scraped_configs时候也有体现.
如果这两个标签已经存在于scraped data里，这时候取决于 配置项honor_labels。

honor_labels = true: 将保存scraped data里的数据 忽略 server端labels
false： 通过重命名来解决，将scraped data  标记为 "exported_\<original-label\>",然后添加上server端的labels

server端配置的external_label不受此设置影响，external_label只适用于time series没有给定的labels，如果有，就被忽略。

## querying
prometheus 提供一个 强大的查询语言来为用户提供实时的查询和聚合 time series 。
prometheus 的 expression language 可以操作4种类型：
1. instant vector 一组time series 包含每个time series 的一个 sample，共享同一个时间戳
2. range vector   一组 time series 包含随时间变化的  每个 time series 的 data point值。
3. scalar 简单的数字浮点值
4. string 字符串值，目前未使用

举例子说明吧：
instant vector:  node_load1 表达式得出的结果 （同一个时间戳）
range vector: node_load1[5m] 表达式得出的结果 多个时间戳（间隔就是 配置里的抓取间隔）

操作符：
算术操作符：
+ - * ／ %  ^
比较操作符：
==  != > < >= <=
逻辑操作符：
and or unless
Aggregation operators:
sum min max avg stddev stdvar count count_valued bottomk topk quantile
优先级：
^
*，/，%
+， -
==，!=，<=，<，>=，>
and， unless
or


## recording rules
recoding rules 可以帮助你将经常需要计算，或者计算很消耗性能的 表达式 预先计算并保存为新的 time series 然后查询recoding rules 会比每次执行原始表达快得多，这对于dashboard有用，每次dashboard刷新时都会去查询相同的表达式。
语法：
> \<new time series name\>[{\<label overrides\>}] = \<expression to record\>

放点简单的配置吧：
```
cat recoding.rules
node_host = sum(label_replace(node_boot_time, "host", "$1", "instance", `([\d\.]+):[\d\.]+`)) by (host)
```
上边这个例子在表达式里使用了 Aggregation  Function ，赋值给 node_host 这个metric name。

另外这这里也说一下prometheus 的配置文件里支持抓取时对labels的处理。添加或者根据正则表达式替换，非常方便。
举个例子，
prometheus 抓取时 会有个instance的标签 base on __address__ 这个tmp标签，格式时 ip：port
那我在后边用的时候不想用这个port 直接用 ip来显示。
```
- job_name: 'node'
    scrape_interval: 5s
    static_configs:
      - targets: ['192.168.30.200:31083','192.168.30.201:31083','192.168.30.209:31083','192.168.30.210:31083','192.168.30.211:31083','192.168.30.214:31083','192.168.30.215:31083', '192.168.30.239:31083', '192.168.30.195:31083', '192.168.30.202:31083', '192.168.30.203:31083', '192.168.30.108:31083', '192.168.30.156:31083']
        labels:
           node_text_label: 'node_scarpe_label_config'
    relabel_configs:
      -  source_labels: [__address__]
         target_label: node_host
         regex: ([\d\.]+):[\d\.]+
```
即国土：
![](http://opiq5jspn.bkt.clouddn.com/prometheus.png)
这个例子最后就可以给你的series填上你自定义的标签。

## 配置
prometheus 由2部分配置组成，一个是command-line flag 和 configuration file。command-line flag 是来配置不可变的系统参数比如说storage location。 configuration file 用来定义去 scraping 的 job instance 和 一堆rules
prometheus -h 是可以看当前版本可用的avaliable command-line。
configuration file 的配置可以通过HTTP POST 来随时reload， yaml格式，官网针对每个配置项都有详细的说明可以在使用中探索。
command-line flag 大致分为5个部分：
1. ALertmanger的一些简单配置
2. Log 的简单配置
3. query 的简单配置
4. storage 大量的配置
5. web的简单配置

这里主要总结下 storage的配置：
prometheus 有自己定规的存储layer 用恒大大小1024字节的chunk来组织采样数据，然后这些chunks都是根据每一个time series 一个文件存放在磁盘上。

内存：
在v1.6.0之后 storage.local.target-heap-size 替代了 原来storage.local.memory-chunks和storage.local.max-chunks-to-persist。保留了选项但是默认值都是0。只有 storage.local.memory-chunks 非0 时 覆盖 heap-size 的值为 x * 3072
官方建议 storage.local.target-heap-size 的值为物理内存的三分之二。
官方默认的GOGC是40%，如果你想修改可以覆盖环境变量

磁盘：

storage.local.path 存储使用目录
storage.local.retention 采样数据保留多长时间，默认360小时

规划 prometheus 的磁盘使用容量 可以简单利用一个计算公式

needed_disk_space = retention_time_seconds * ingested_samples_per_second * bytes_per_sample

每个smaple大约是1-2个字节
在生产环节也验证了这个值是差不多的。

chunk encoding:

storage.local.chunk-encoding-version  0,1,2 三个版本
默认是1，1的压缩比0好，0是最快的，官方推荐1.
0，1都是由固定位数的bit-width。  2 是 可变的bit-width 的encoding。
2提供更高的压缩比，但是cpu和查询的延迟会增大。
官方有blog对比!()[https://prometheus.io/blog/2016/05/08/when-to-use-varbit-chunks/]有兴趣可以看看。

alertmanager:

之前弄邮件报警的时候有个坑：
qq邮箱默认的发送服务器：
smtp.exmail.qq.com(使用SSL，端口号465)
而prometheus 发送邮件使用的 STARTTLS  ，具体没有了解过这2个协议的区别，注意下就好
smarthost: 'smtp.exmail.qq.com:587'




##### 解决方案资源：
https://stefanprodan.com/2016/a-monitoring-solution-for-docker-hosts-containers-and-containerized-services/





















