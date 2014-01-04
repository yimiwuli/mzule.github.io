title:Solr性能优化
time:2014-01-04 19:33:10
template:post.html
---
> 本文主要转载自知乎的问题[Solr，Lucene 优化有哪些相关经验可分享?](http://www.zhihu.com/question/19849847) [清尘](http://www.zhihu.com/people/qingchenzhuoshui)的答案

(对于Solr优化)倒是没有太多的经验， 不过最近着手做这方面的东西， 希望能讨论讨论：

你的索引文件30G！这可是个非常大的数目了。

### 1. 从schema入手：

1. 你的数据是都需要store的么， 比如一些字段只需要提供结果搜索而不作为结果返回， 那么需要将store设置为false从而减少数据量。
2. 你的所有字段都需要highlight么， 如果不需要highlight则可以将term vector关掉。 
3. 确认你在schema没保存太多无用的信息（即不用来搜索也不用来返回）。
4. 你的分词算法是否合理， 如果你使用的是ngram的分词方法， 可以通过设置最大和最小分词长度来限制分词数据。
5. 你的业务上是否有明显的条目信息， 比如你索引的东西是一本书， 很多其他的信息在mysql或者其他地方有副本， 这样的话， 你大可以只store id信息， 其他字段都不store。
6. 你的业务能不能拆分， 比如之前书影音一起做搜索， 你可以考虑将他们分开。

----

### X. 自己的补充：


1. 对于不需要进行搜索的字段，设置index="false"。
2. 移除没有必要的copyField配置。
3. 为了达到最少的索引大小和最佳的搜索性能，可以考虑，设置所有的文本类型的字段index="false"，并使用copyField将所有的文本字段copy到一个单独的字段，用这个字段进行搜索。
4. 在程序中使用`StreamingUpdateSolrServer`可以最大化建立索引的性能。
5. 记住以server mode启动JVM，并使用较高的log level，以减少日志的输出。

---

### 2. 从segment角度：

1. lucence使用segment来保存索引数据， 可以在Solr中通过mergeFactor来指定需要的临时文件数目， 如果mergeFactor过大， 则index的时间会缩短， 但磁盘占用和搜索时间会变的很长， 可以考虑适当调整mergeFactor， 或者在slave上做定期的optimize操作， optimize会将segment合并到一起， 当然， 这个过程比较耗资源， 所以可以考虑optimize的时候指定要merge到的数据。
2. 同样， 在master ， slave环境中， 最好不要在master上做optimize， 因为在master上做optimize会导致slave每次复制整个index的数据， 这当然不是你想要的， 所以要采取措施不要让master和slave的配置一样， 这样很不划算。
3. 从shard上，更一般的情况是， 如果这些都不是你想要的， 可以考虑shard， Solr支持很灵活的Shard， 你可以在build index的时候给出规则让不同的数据发送到不同的服务器， 比如同过hash等。 这样你的大数据就可以分布在多个机器中， 提高处理速度， 当然， 如果你有性能较好的机器， 同样可以考虑在单一机器上做shard， 利用多核的处理性能。但是要谨慎Solr的一些特性在Shard环境中的支持情况。
4. JDK, JDK 的版本， 建议使用最新的版本， 但是7貌似有bug， 看有没有7.1， 32 vs 64 对java没影响， 都是byte code ，不区分。可以通过制定Xmx Xms 参数调整heap 的大小， 如果你经常出现heap over flow的话。启用多线程gc， 这是你在处理大数据时避免溢出的好方法， 一般一个core一个gc线程。
5. 设置适当的cache， 根据cache命中率的不同来选择fastlru or lru。这里网上有个叫做chenlb的人以前在1.4中做了一个patch， 可以使用memcache做solr的cache， 不过现在的3.5版本不兼容， 作者也没有对其更新， 如果你在分布式环境下， 可以考虑自己做一下这个， 弄好了可以分享给笔者。
6. 根据使用的不同， 优化方式也非常的多， 不再赘述。

