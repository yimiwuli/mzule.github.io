## 提高性能的方式

1. 对于只需要进行搜索，不需要返回具体内容的字段，设置stored="false"，尤其是一些内容较大的字段。
2. 对于不需要进行搜索的字段，设置index="false"。
3. 移除没有必要的copyField配置。
4. 为了达到最少的索引大小和最佳的搜索性能，可以考虑，设置所有的文本类型的字段index="false"，并使用copyField将所有的文本字段copy到一个单独的字段，用这个字段进行搜索。
5. 在程序中使用StreamingUpdateSolrServer可以最大化建立索引的性能。
6. 记住以server mode启动JVM，并使用较高的log level，以减少日志的输出。

## shecma

1. **name**: 给这个schema.xml起一个名字，仅仅用于显示，没有其他作用。
2. **version**: 定义了这个schema.xml的对应的schema语法版本，不同的版本对默认值的处理方式不一样。

##field

1. **name**: 字段的名称。以`_`开头并以`_`结尾的名称都是SOLR的保留名称。例如`_version_`
2. **type**: 字段的类型。
2. **indexed**: 是否对字段建立索引，建立索引之后才可以使用该字段进行搜索和排序。
2. **stored**: 是否存储，只有设置为true的时候，在搜索时，该字段才可以展示出来。
2. **docValues**: Solr4.2之后才有的一个选项，设置是否对该字段建立forward index(和inverte index相反，以达到互补的目的). DocValue对于执行facet,group,sort以及function 2. query是有利的，而且，docValue可以加快索引的加载速度，更好的NRT支持，docValue还可以节省内存的使用。但是，DocValue目前只支持StrField, UUIDField 和Trie*Field, 对于某些类型的数据，可能还会要求数据不能是multi-valued的以及必须要有值。详情参考：[http://wiki.apache.org/solr/DocValues](http://wiki.apache.org/solr/DocValues)
3. **multiValued**: 设置该字段是否可以包含多个值。
4. **omitNorms**: 设置该字段是否忽略norms，设置成true可以节省部分内存和硬盘空间，同时也关闭了文档的长度归一化和索引阶段的字段加权功能。只有当字段需要加权，或者字段需要支持全文检索的时候，才需要norms。默认的，对于所有的基本类型的字段，该属性默认值为true.
5. **termVectors**: 设置是否保保存字段的词向量模型，当使用MoreLikeThis的功能的时候，设置该字段为true可以提升性能。
6. **termPositions**: 设置是否在词向量模型中保存位置信息，保存位置信息会增大索引的体积，但是可以用于hightlighting.
7. **termOffsets**: 设置是否保存在词向量模型中保存offsets信息，同样的，这样做会增大索引体积。
8. **required**: 设置该字段是否必须要有值，相当于DB的 not null.
9. **default**: 设置字段的默认值。

## dynamicField

dynamicField的名称中的*只能有一个，要么在开头，要么在结尾。

使用下面的配置可以忽略一切在schema.xml中没有配置的字段，而不是抛错。

```
<dynamicField name="*" type="ignored" multiValued="true" />
<fieldType name="ignored" stored="false" indexed="false" multiValued="true" class="solr.StrField" />
```

## fieldType

## others
