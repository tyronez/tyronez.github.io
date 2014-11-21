---
layout: post
title: "CMU神课全解析——15619 Cloud Computing"
description: "初来乍到CMU，第一个学期对我的世界观造成巨大影响的一门课：云计算（Cloud Computing）——原来课还能这样上！"
category: 
tags: []
---

早上起来，第一件事就是打开Cloud Computing的课程网站，看看我们小组搭的Web Service Cluster在昨天晚上长达四个小时的Performance Live Test得到的跑分——结果有点差强人意，不过和之前预估的情况基本吻合。这次Live Test的结束也标志着这个学期这门有时超级坑爹、有时超级有意思的课终于走到了尾声。

Cloud Computing(15319/15619)，是CMU SCS（School of Computer Science）开的一门面向全校本科生和研究生的课程，主要内容顾名思义：云计算。之所以我称之为神课，一方面是由于课程形式：一切教学都在网上进行。教学视频放到网站，供CMU跨好几大洲的不同校区的学生观看；定期进行Online Quiz；Weekly Project自己写代码跑实验，结果提交到Online Judge System；Semester Project则是三人一组搭建Web Service，提交地址到指定的测试网站，由测试系统自动测试性能和正确性；另一方面是，这门课帮你报销，带你玩转AWS（Amazon Web Service），土豪不差钱的节奏（其实还是有限制的，所以每次project都会告诉你不能用超budget，否则扣分，严重的劝退。我自己因为超budget被请喝茶过一次，据说还有人一次超了300+刀直接被劝退了）；至于最后一个原因，就是：坑很多，水很深。

闲话少说直接开始介绍课程内容。正如前面所说，主要分为三个部分：Online Quiz，Weekly Project，Semester Project。

**Online Quiz**
这部分每次主要是给一些阅读材料，介绍云计算中的某些Topic，然后进行开卷的在线答题，题目主要是前面阅读材料涉及到的概念、公式、Case Study之类，限时2到3小时不等。网站自身提供搜索功能，所以根据题目找对应的内容会相对容易一点——尽管如此我还是没拿满过，那感觉就跟在做GRE语文＋数学的混合进阶版一样。这部分能让学生广泛接触云计算的方方面面，比如发展历史、Data Center的搭建，比如Virtualization, Cloud Storage如何Scale Up，再比如分布式计算（MapReduce），把材料都完整看下来的话虽然每个领域都不会深入，但是起码会对整个方向有个大体的认识。

**Weekly Project**
这门课基本每周会出一个小Project，形式大概都是自己部署AWS的资源跑实验，手动配置或者写代码自动跑，然后把实验结果提交到网上。至于具体的内容就千奇百怪无奇不有了，我举几个栗子：有一个Project Group，利用AWS的ELB（Elastic Load Balancer）和ASG（Auto Scaling Group）进行负载均衡和根据当前访问压力自动Scale up／down，要命的是做实验尝试不同配置，而每次实验要跑40分钟（后来还有一个类似的Project，一次跑100分钟），还要小心翼翼控制budget（后来我告诉自己，其实助教设定这样的时间是用心良苦啊，40分钟的实验，刚好是一集美剧的时间；100分钟的实验，恰好是一部电影的时间。于是深谙助教的用意的我打开了搜狐视频……）；另一个例子是云存储的对比，文件系统/MySQL/Hbase，也是跑给定的benchmark然后比较性能，这里面涉及到2小时教你精通Bash Script，手把手教你配置MySQL Cluster之类的，不是很有意思，但也能在做实验的过程中学到不少东西；比较有趣的是用MapReduce写程序构建statistic n-gram model，支持搜索的自动补全，不过这个Project还没完全放出，在此略过。

**Semester Project**
这个Project可以说是这门课的精髓，内容是搭建一个Twitter Analysis的Web Service。具体而言，在给定的数据集上（1TB的大小），支持下列的查询

Q1: 用作Heartbeat，其实就只是算一个大整数除法  
Q2: 给定用户ID和时间戳，返回这个用户在这个时间戳发的所有推文  
Q3: 给定用户ID，返回所有转发过这个用户推文的所有用户（包括自己，去重，并且把互相转发——也就是这个被转发的用户也转发过这个转发者的推文——的情况标记出来）的ID  
Q4: 给定日期和地点和一个范围（m到n），返回这个日期和地点的hashtag列表，按热门程序也就是推文数排序，取第m个到第n个。  
Q5: 返回每个用户的意见指数（我随便起的名字大家意会即可）。每个用户的意见指数计算如下：每发一条推文，＋1；每当有推文被转发一次，＋3；每有一个用户转发过其推文，＋10（也就是A转发B的推文两次，A也只是一个用户，＋10）  
Q6: 给定一个用户ID的范围（m到n），返回所有ID在这个范围内的用户发过的照片总数

这个Project的实现涉及到方方面面，但主要可以分为下面三块：

1. 数据库设计。合理的数据库设计才能保证查询效率。因为只有读操作而没有写操作，所以这里面可以做很多trick，包括存一整个处理好的字符串、设计一个exactly match的hbase row key之类的。还有就是，cluster还是单机数据库？我们选的是纯单机，因为cluster对于纯读的场景来说效率反而差（无论是mysql还是hbase都是如此）。    
2. ETL（Extract-Transform-Load)。好吧说白了就是导数据，说ETL那是在伪装高端。这部分主要是通过MapReduce导出需要的数据，有一些不大不小的坑，都是一些细节，一不小心出来的数据就是错误的。给的twitter数据集包含重复的推文，因为这点导致本来可以一次MapReduce跑完的Job需要拆分成两个不同的Job。我做完这个部分之后才真正体验到MapReduce是个好东西，一个小时处理完所有数据完全无压力——如果是用Java写的话。Well……我是Python脑残粉＋Java黑，但是不可否认的是几乎所有用Python写的同学都表示用Python跑的时间起码是Java的好几倍。。。  
3. Web Service。这部分主要是搭建响应请求的服务器（这里叫Front End，注意和HTML＋JS前端不是一个东西），逻辑不算很复杂，主要是从数据库读处理好的数据然后返回，但是也很challenging，因为好的Front End对throughput影响很大——数据库连接池、资源释放、数据的内存Cache，还有自己用Nginx做Load Balancer（因为AWS的ELB实在是不靠谱），工作量还是挺大的。

然后谈谈测试。正如前面所说，测试是先搭好一个Web Service，然后把URL提交到测试网站上面，这个网站就会自动schedule一个测试任务，在给定的时间内不断产生特定类型的请求到这个URL，验证正确性和性能。至于Live Test也差不多，只不过测试周期长达数小时，不同类型的请求混合出现。不过这里面有一个设计不太合理的地方是throughput占的比重太高，以至于即使correctness只有80+也能靠刷吞吐量拿满分。除了这个原因和只能看不能点的“取消测试”按钮之外，整个测试系统还是让人比较满意的。

总结一下这个课带给我的几点收获和影响：  
1. 对云计算涉及到的方方面面有了基本了解  
2. 每周一次的小Project算是给我培养了一些运帷的素养  
3. 让我开始拥抱Java（好吧这真是出乎我的意料之外）和MapReduce  
4. 培养了目光检视找BUG的能力（跑一次时间久还烧钱所以试错调试成本太高）  
5. 培养了budget意识，一生受用（不这么说都对不起Majd那杯热茶）  

最后对从头到尾碾压Project的FDU小组表示致敬(知道这个名字真相的我泪流满面)

<img src="/images/fdu.png">






