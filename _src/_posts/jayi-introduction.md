title:这个博客是基于Jayi搭建的
template:post.html
---
##1. 诞生记
以前一直用`csdn`或者`cnblogs`的博客服务，虽然说使用上还过得去，但是总是觉得不够geek. 自己有考虑过购买虚拟空间，还试用了几家，后来因为种种原因而一直耽误到今天。 意外的发现`github`提供了`pages`功能，并且鼓励用户利用该功能搭建博客、开源项目的官网。顿时觉得天上掉下了馅饼。哈哈~

按照阮一峰老师的[教程](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)一步一步的完成了一个Hello World之后，开始正式的折腾了，发现`Jekyll`怎么都装不了，网上也找不到相关解答，`ruby`自己也不熟悉。顿时怒了，决定自己写一个静态网页生成器，于是`Jayi`就诞生了。`Jayi`是一款静态网页生成器，非常适合在github pages服务上搭建个人博客。

##2. 使用Jayi
###2.1 快速开始
Fork me at [github](https://github.com/mzule/mzule.github.io)
###2.2 内建变量 
Jayi的内建变量包括：

1. **title**
2. **link**
3. **time**
4. **content**
5. **next_post_title**
6. **next_post_link**
7. **previous_post_title**
8. **previous_post_link**

###2.3 自定义变量




##3. Jayi的设计
在设计Jayi的时候，我首要考虑遵循KISS原则，保持简单，可以约定的事情，就不用去配置。于是，在Jayi中就有了很多约定。如下：

###3.1 目录结构
jayi项目约定的目录格式：

```
project // Jayi项目
    +-----_src// 项目的源码
            +---_includes// 公共文件目录，比如 menu.html/footer.html等等
            +---_templates// 模板文件目录
            +---_posts// 存放博文的目录
            +---index.html// 首页
    +----// 项目生成的静态网页代码
```
案例：
```
mzule.github.io
    +-----_src
            +---_includes
                    +--- header.html
                    +--- footer.html
            +---_templates
                    +--- post.html
            +---_posts
                    +--- Hello World.md
            +---index.html
            +---css
                   +--- style.css
```

之所以把生成的目标文件放在外层目录，而源码文件被放到了_src目录，是因为这样做可以直接把源码和网页同意通过github的项目管理起来。

###3.2 处理流程
####3.2.1 PostProcessor
Post是Jayi中的一个基本元素，代表着一篇博文，可以是Markdown的格式，也可以是html的格式。Post全部存放在_src/_posts/目录下。

原始的Post在Jayi中称作RawPost，Jayi对Post的处理，就是RawPost进化到Post的一个过程。Jayi采用松散的设计，对Post的处理，是通过一个个单独的PostProcessor对象来进行的。当前Jayi实现的PostProcessor包含：

1. TargetFileNameProcessor
2. DefaultKeyValueProcessor
3. KeyValueProcessor
4. MarkdownProcessor
5. SummaryProcessor
6. TemplateProcessor
7. VariableProcessor

他们都实现了PostProcessor接口，在对Post的处理流程中，可以被轻松地添加和删除。日后需要新的添加新的处理方式，只需要实现新的PostProcessor接口，然后注册到PostCompiler中即可。

####3.2.2 PostListProcessor

上面说的 PostProcessor是单独的对一个个Post进行处理，而PostListProcessor是站在全局的角度，对所有的Post进行处理。现在主要包含两个实现：
1. OrderProcessor
2. PreNextPostProcessor
未来可以想到的还有SimilarPostProcessor等等，等以后有需要了也可以很方便的添加进去。

####3.2.3 PostCompiler
PostCompiler负责整合PostProcessor和PostListProcessor的处理流程。分成三个阶段：

1. BeforeListProcess
2. ListProcess
3. AfterListProcess

####3.2.4 IndexProcessor

IndexProcessor是对index.html进行处理的处理器，只负责这一个页面的处理。主要的任务负责把所有的Post通过列表的方式展示在index.html中。

####3.2.5 Main
Main是整个程序的入口，处理流程包含三个阶段：

1. PostCompiler
2. IndexProcessor
3. CopyStaticResources


