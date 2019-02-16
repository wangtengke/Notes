
# 学习计划

学习时间 | 学习内容| 备注
---|---|---
第一周 (11.26)| 1.学习java集合，hashmap和concurrenthashmap在1.7和1.8的区别<br>2.学习java并发相关知识，synchronized和volatile关键字<br>3.看java虚拟机的书 |写项目的串口程序
第二周 (12.3)| 1.看activemq，了解queue和topic两种消息传输方式，queue为队列传输，topic为订阅发布<br>2.学习disruptor并发框架，cas无锁操作<br>3.复习+学习netty和rpc框架(正在学习中)，了解netty传输机制，rpc远程调用，服务注册与消费<br>学习springboot+jpa | rpc框架还没学完，需要理解远程代理，争取周末搞定
第三周 (12.10)| 1.看java虚拟机，了解gc收集器，类加载机制<br>2.刷leetcode题 | 需要准备下面经上的东西
第四周 (12.17)| 1.看数据库相关，写了redis的五种数据类型的操作demo，了解了mysql的B+树原理，磁盘读写原理<br>2.刷leetcode题 |待学习知识，计算机网络，操作系统
第五周 (12.24)| 1.刷leetcode题，了解一些基本算法思想 | 需要系统的过一遍各种常见算法，比如dp
第六周 (12.31)| 1.看ceph相关知识，准备了解分布式存储的一些知识，算法以及理论，了解对象存储，块存储，文件存储相关知识 | 分布式云存储是云计算的重要环节，值得深入研究
第七周 (1.7)| 1.了解ceph工作原理,了解ceph内部架构和存储原理<br>2.熟悉linux ceph相关指令,熟悉 ceph client monitor osd相关指令，简单进行操作<br>3.更新监控脚本,兼容不同ceph版本pg节点数据监控的字段不同问题 | ceph相关工作
第八周 (1.14)| 1.ceph源码编译,升级centos版本至7.4,编译ceph源码,编译rpm包<br> 2.ceph pool级别监控、报警(0->50%),pool中的iops和流量数据的提取,pool级别报警 | ceph相关工作
第九周 (1.21)| 1.ceph集群内互ping(0->30%),集群内mon节点互ping,集群内osd节点互ping,osd节点ping mon节点<br>2.ceph pool级别监控、报警(0->100%),pool中的iops和流量数据的提取,pool级别报警 | ceph相关工作
第十周 (1.28)| 1.ceph集群内互ping(0->80%) 开发完成，待在集群中测试<br> 2.cosbench压测(0->50%) 测试ceph集群的压力，目前客户端压力还是不够，需要加大 |ceph相关工作
第十一周 (2.4)| 春节休息 | 无
第十二周 (2.11)| 1.cosbench压测(50->80%)，再次加入3台客户端进行压测，效果不变<br>2. ceph 对象存储一个 bucket 2000万对象大量读写删除操作故障模拟(0->80%),测试结果不会block，待加大删除速度<br>3. snapshot 的实现以及对性能的影响(0->100%),对对象存储有影响，对rbd块存储无影响 | ceph相关工作
第十三周 (2.18)| row 2 col 2 |
第十四周 (2.25)| row 2 col 2 |


---

# 计划学习内容
 
## 算法
- [**剑指offer**](https://note.youdao.com/)
- **Leetcode**
- **算法**
## Java
- [**Java基础**](https://github.com/wangtengke/Notes/blob/master/notes/Java%E5%9F%BA%E7%A1%80.md)
- [**Java集合**](https://github.com/wangtengke/Notes/blob/master/notes/java%E9%9B%86%E5%90%88.md)
- [**Java并发**](https://github.com/wangtengke/Notes/blob/master/notes/java%E5%B9%B6%E5%8F%91.md)
- [**Java虚拟机**](https://github.com/wangtengke/Notes/blob/master/notes/Java%E8%99%9A%E6%8B%9F%E6%9C%BA.md)
- [**Java I/O**](https://github.com/wangtengke/Notes/blob/master/notes/JavaIO.md)
## 数据库
- [**数据库原理**](https://github.com/wangtengke/Notes/blob/master/notes/%E6%95%B0%E6%8D%AE%E5%BA%93%E5%8E%9F%E7%90%86.md)
- **SQL语法**
- **MYSQL**
- **Redis**
## Spring
- **IOC**
- **AOP**
- **MVC**
- **Sringboot**
- **SpringCloud**
## 设计模式
## 网络
- **计算机网络**
- **HTTP**
- **Socket**
## 操作系统
- **Linux**
- **计算机操作系统**
## 框架类
- [**RPC框架**](https://github.com/wangtengke/Notes/blob/master/notes/RPC%E6%A1%86%E6%9E%B6.md)
- [**消息中间件**](https://github.com/wangtengke/Notes/blob/master/notes/%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6.md)
- [**Disruptor并发框架**](https://github.com/wangtengke/Notes/blob/master/notes/Disruptor%E5%B9%B6%E5%8F%91%E6%A1%86%E6%9E%B6.md)
## 测试开发相关
## 后台框架
- [**Django Python web 框架**](http://www.runoob.com/django/django-tutorial.html,https://code.ziqiangxuetang.com/django/django-tutorial.html)
## 前端框架
- **Element UI**
- [**vue**](http://www.runoob.com/vue2/vue-tutorial.html)
- [**jquery js的库**](https://www.runoob.com/jquery/jquery-tutorial.html)
- [**Bootstrap html 样式**](http://www.runoob.com/bootstrap/bootstrap-tutorial.html)
## AI人工智能
- **Darknet框架**
- [**YOLOv3 深度神经网络**](https://pjreddie.com/darknet/yolo/ )
## 服务器部署
- **Linux 远程IP**
- **Docker 镜像容器py2,py3**
