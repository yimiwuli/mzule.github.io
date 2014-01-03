title:解决prettify进行代码高亮导致网页加载慢的问题
time:2014-01-02 19:02:07
template:post.html
---
技术博客都少不了代码高亮，在前端`javascript`进行代码高亮的插件中，`prettify`是一个重要的成员。本博也打算使用的`prettify`进行代码高亮。

下面是官方的`Setup`

```
Download a distribution
Include the script tag below in your document
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js"></script>
See Getting Started to configure that URL with options you need.
Look at the skin gallery and pick styles that suit you.
```

但是在根据官方的`Setup`使用的时候，发现一个难以接受的问题：可能是由于`GreatFw`的原因，引入了`google code`的`prettify`之后，页面加载变得巨慢无比。

尝试手动访问[https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js](https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js)，也是慢的惊奇，果然是新引入的`javascript`导致的页面加载问题。

找到了问题的源头，问题就已经解决了一半。

根据以往的经验，访问谷歌的服务，如果响应很慢，把`https`协议换成`http`协议会使得速度提升很多。试了一下手动访问[http://google-code-prettify.googlecode.com/svn/loader/run_prettify.js](http://google-code-prettify.googlecode.com/svn/loader/run_prettify.js), 嗖的一下，页面加载结束，如沐春风。

于是，赶紧把网页中`prettify`引用的协议改成了`http`，再一次尝试......额(⊙o⊙)…，还是很慢。

后来潜心分析，发现这个`prettify`的文件名叫做：`run_prettify.js`, 肯定是它自己本身也引用了其他的`javascript`文件，打开看一下，果然逮住了3个`https`协议的地址。

好吧，在本地存一份`run_prettify.js`，把里面的`https`统统改成`http`.  然后再网页中引用本地的`run_perttify.js`. 再次运行博客。飞一般感觉。


