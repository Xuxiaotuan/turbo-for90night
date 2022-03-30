## Introduction - Day 1 

今天主要看了Java的SPI机制,以及Mysql的文件排序
### SPI
> SPI（Service Provider Interface），是JDK内置的一种 服务提供发现机制
#### SPI与API区别：
API是调用并用于实现目标的类、接口、方法等的描述；
SPI是扩展和实现以实现目标的类、接口、方法等的描述；
换句话说，API 为操作提供特定的类、方法，SPI 通过操作来符合特定的类、方法。
#### 应用场景: 
Common-Logging，JDBC，Dubbo等等
#### SPI流程：
1. 有关组织和公式定义接口标准
2. 第三方提供具体实现: 实现具体方法, 配置 META-INF/services/${interface_name} 文件
3. 开发者使用

#### 不足
1.不能按需加载，需要遍历所有的实现，并实例化，然后在循环中才能找到我们需要的实现。如果不想用某些实现类，或者某些类实例化很耗时，它也被载入并实例化了，这就造成了浪费。
2.获取某个实现类的方式不够灵活，只能通过 Iterator 形式获取，不能根据某个参数来获取对应的实现类。
3.多个并发多线程使用 ServiceLoader 类的实例是不安全的。

### MySQL Order By 文件排序
1. 整体概览
2. 排序缓冲区（sort buffer）
3. 单个排序字段太长怎么办？
4. 排序模式  
    4.1 <sort_key, additional_fields>  
    4.2 <sort_key, packed_additional_fields>  
    4.3 <sort_key, rowid>
5. 提升排序效率  
    5.1 优先队列  
    5.2 随机 IO 变为顺序 IO
6. 两类排序  
    6.1 内部排序  
    6.2 外部排序
7. 倒序排序
8. 窥探更多排序细节
9. 总结


**引用文中的总结如下**
>介绍了系统变量 **sort_buffer_size** 用于控制排序缓冲区大小，**max_sort_length** 用于控制单个排序字段内容长度。
>
>**排序模式小节**，介绍了三种排序模式：
>
><sort_key, additional_fields> 把排序字段（sort_key）和存储引擎返回给 server 层的字段都写入排序缓冲区或磁盘文件，每个字段都以其最大长度占用存储空间，存在在空间浪费。
><sort_key, packed_additional_fields> 是 <sort_key, additional_fields> 的改进版，解决了空间浪费的问题。char、varchar 字段只占用实际内容需要的空间，内容为 NULL 的字段除了在 NULL 标记区域占用 1 bit 之外，不会再占用其它空间。
> 
>前面两种排序模式，以空间换时间，虽然不需要**两次**访问存储引擎，让文件排序逻辑整体上更简单，但是记录数量多起来之后，需要磁盘文件存储排序结果，而磁盘 IO 会导致排序效率低下。
><sort_key, rowid> 需要两次访问存储引擎，但只写入排序字段（sort_key）和主键 ID 到排序缓冲区，一定程度上避免了使用磁盘文件存放排序结果，某些情况下可能会比前两种排序模式更优。
>**提升排序效率**小节，介绍了源码中采用的两个优化方案：
>
>SQL 语句中包含 limit 的情况下，通过成本评估有可能会使用优先队列来避免磁盘文件排序，提升排序效率。
>使用 <sort_key, rowid> 排序模式，并且由于记录数量太多需要使用磁盘文件时，通过把主键 ID 缓存在随机读缓冲区（read rnd bufer），缓冲区满后对主键 ID 排序，再通过主键 ID 去存储引擎读取客户端需要的字段，尽可能把随机 IO 变为顺序 IO，提升排序效率。
> 
>**两类排序**小节，介绍了三种内部排序算法，其使用优先级为：基数排序、快速排序、归并排序。
>
>外部排序，可能会进行 0 ~ N 轮归并排序生成**中间结果**，最后进行一轮归并排序得到**最终结果**。
>
>生成中间结果的归并排序，排序字段（sort_key）也会写入到磁盘文件，后续生成中间结果和最终结果的归并排序都会用到。生成最终结果的归并排序，磁盘文件只写入存储引擎返回给 server 层的字段，不会包含排序字段（sort_key）。
>
>**倒序排序**小节，介绍了倒序排序的实现：先对排序字段（sort_key）逐字节取反，然后对排序字段进行正序排序，最终得到倒序排序的记录。
>
>最后，介绍了如何通过 optimizer trace 窥探文件排序的更多细节

### 已读文章
- [x] [带你一步一步深入了解 MySQL Order By 文件排序](https://mp.weixin.qq.com/s/YBWbEahWp0uVN_n9jAFU0A)
- [x] [MySQL 查询语句的 limit, offset 是怎么实现的？](https://mp.weixin.qq.com/s?__biz=Mzg3Njc0NzUwNQ==&mid=2247483766&idx=1&sn=cb61d035c97123e6308b1a2e41ea447c&chksm=cf2ccee8f85b47fe2e60decfac2ca039ec74430fee821a9b3ee2e6b8372cff71700a2856a86b&scene=21#wechat_redirect)
- [x] [Mysql 分组计汇, MySQL中with rollup的用法, group by汇总](https://justcode.ikeepstudying.com/2019/03/mysql-%E5%88%86%E7%BB%84%E8%AE%A1%E6%B1%87-mysql%E4%B8%ADwith-rollup%E7%9A%84%E7%94%A8%E6%B3%95-group-by%E6%B1%87%E6%80%BB/)
- [x] [设计原则：小议 SPI 和 API](https://www.cnblogs.com/happyframework/p/3325560.html)
- [x] [Java SPI思想梳理](https://zhuanlan.zhihu.com/p/28909673)
- [x] [springboot中SPI机制](https://www.jianshu.com/p/0d196ad23915)
- [x] [深入理解 Java 中 SPI 机制](http://blog.itpub.net/69912579/viewspace-2656555/) 
- [x] [彻底搞懂Class.getResource和ClassLoader.getResource的区别和底层原理](https://blog.csdn.net/zhangshk_/article/details/82704010)
- [x] [commons-logging实现日志解耦](https://blog.csdn.net/sakurainluojia/article/details/53534949)
- [x] [Difference between SPI and API?](https://stackoverflow.com/questions/2954372/difference-between-spi-and-api)
- [x] [Java transient关键字使用小记](https://www.cnblogs.com/lanxuezaipiao/p/3369962.html)
- [x] [Java常用机制 - SPI机制详解](https://www.pdai.tech/md/java/advanced/java-advanced-spi.html)
