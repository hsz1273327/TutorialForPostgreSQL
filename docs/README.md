# 使用postgresql做数据存储

postgresql(简称pg)是目前最先进的开源关系型通用数据库,它完全支持标准sql语句,在其上还有一定的功能扩展;天生支持插件,有许多额外的实用功能由一些第三方组织已经做了很不错的实现;并且支持自定义数据包装器,将pg仅作为interface,数据则实际存在被包装的数据库上.

在性能上,默认配置下的pg从来不在任何一个场景下是最好的选择.但永远是第二好的选择,加上它是个多面手,可以一套工具应付绝大多数场景,不用考虑多种工机的异构系统集成问题,因此技术选型选它还是比较合适的.

postgresql的上限极高,单实例基本可以认为给多少资源都能用满.适当的资源和设置下可以达到各个领域一线的性能水平.但下限也很低,配置不当很容易让人认为它不适用于当前场景.而优化配置pg需要相当的学习成本.我们基本可以认为pg是开源版本的oracle.正因为如此许多传统软件领域比如银行交易系统,erp系统等会用它替代oracle,而国内互联网企业普遍不用它而转向设计更加粗糙但学习成本相对低的mysql的原因.毕竟互联网企业相对业务简单,而且生命周期短,一般业务设计也不严谨,不太愿意花钱请dba.

## 特有功能



+ 缓存: 物化视图

+ 搜索引擎: 全文搜索索引足以应对简单场景;配合插件zhparser,jieba等分词工具可以实现中文支持,丰富的索引类型,支持函数索引,条件索引

+ 文档数据库: JSON,JSONB,XML,HStore原生支持,可以替代mongodb

+ 作为交互界面包装外部数据: 可以自定FDW包装外部数据,也就是说pg可以什么也不存,只是作为对其他外部数据对象的sql交互界面

+ Notify-Listen，在没有消息队列的情况下可以拿他凑活,非常适合快速原型实现

+ 物化视图,将查询固化保存,提高查询效率

+ 存储过程与函数编写

除此之外还有通过插件实现的功能,包括

+ 空间数据: [PostGIS扩展(杀手锏)](https://github.com/postgis/postgis),内建的几何类型支持,GiST索引.

+ 时序数据:[timescaledb时序数据库插件](https://github.com/timescale/timescaledb),支持连续聚合等流式处理.

+ 图数据库: 递归查询,更有[Apache AGE扩展](https://github.com/apache/incubator-age)实现完整的图数据库功能.


本文会按顺序逐次介绍

## 应用领域

pg生态下大致的应用领域有:

+ [OLTP](https://baike.baidu.com/item/OLTP/5019563?fr=aladdin): 事务处理是PostgreSQL的本行,完整事务支持,同时支持行级锁,表级锁,页锁,预锁,应用锁,自旋锁,共享锁,排他锁,丰富的锁类型为复杂事务的应用提供了便利,多种类型的索引,分区表,优化器,以及并行计算能力(CPU/GPU)更是扩展了OLTP业务的边界,一些原本只能离线计算后加载到业务缓存的数据服务也可以使用pg达到实时查询.

+ [OLAP](https://baike.baidu.com/item/%E8%81%94%E6%9C%BA%E5%88%86%E6%9E%90%E5%A4%84%E7%90%86/423874?fromtitle=OLAP&fromid=1049009&fr=aladdin): 并行计算,分区表,ANSI SQL兼容,窗口函数,CTE,CUBE等高级分析功能,同时支持多种语言写UDF,.甚至可以通过插件实现定时触发任务.如果你的OLAP需求处理的数据量在TB级别,不要犹豫pg单机(请适当加资源,上主备)就可以,PB级别可以使用pg体系下分布式方案的[Greenplum](https://github.com/greenplum-db/gpdb),再大的恐怕就还是交给hadoop+spark集群吧.



## 本文使用的工具

+ 本文使用docker作为pg的部署工具,我会在docker中模拟pg和插件的运行环境,我使用的镜像为[hsz1273327/pg-allinone](https://github.com/Basic-Components/pg-allinone)

+ 本文使用的pg版本为`11`,因为

+ SQL语句,基本不用不合规范的SQL语句

+ 执行SQL语句这边使用jupyter notebook的[postgresql_kernel](https://github.com/Python-Tools/postgresql_kernel)

## Helloworld

按照惯例,我们来先写个helloworld,在安装好pg后,默认会有一个数据库`postgres`,我们可以进入它来实现这个helloworld


```sql
-- connection: postgres://postgres:postgres@localhost:5432/postgres
```


```sql
-- autocommit: true
```

    committed current transaction &  switched autocommit mode to True


```sql
select 'hello world' as welcome
```

    1 row(s) returned.





<table>
<thead>
<tr><th>welcome    </th></tr>
</thead>
<tbody>
<tr><td>hello world</td></tr>
</tbody>
</table>



## 设置测试仓库

我们创建一个test仓库用于测试


```sql
CREATE DATABASE test
```
