# 插件扩展

PostgreSQL被设计成很容易扩展,由于这个原因,被载入到数据库中的扩展可以像内建特性那样运行.源码库中的`contrib/`目录中包含了多个扩展的源码,我们前文介绍的`pg_trgm`便在其中.而更有许多优质的第三方扩展,他们的开发是独立进行的,例如`PostGIS`以及前面介绍的分词扩展`pg_jieba`.

pg的扩展一般是3个部分组成:

+ 一个动态链接库用于实现功能,放在`pg_config  --pkglibdir`指定的目录下
+ `extension_name.control`控制文件,声明该扩展的基础信息,放在`pg_config --sharedir`指定的目录下
+ `extension--version.sql`加载扩展所需要执行的SQL文件,放在`pg_config --sharedir`指定的目录下


在配置项`shared_preload_libraries`中可以指定需要预先加载的动态链接库,而使用扩展则需要链接上服务器后使用[CREATE EXTENSION](http://postgres.cn/docs/12/sql-createextension.html)语句:

```sql
CREATE EXTENSION IF NOT EXISTS extension_name
```
