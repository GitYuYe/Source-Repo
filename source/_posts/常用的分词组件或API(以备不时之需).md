---
title: 常用的分词组件或API(以备不时之需)
date: 2016-04-17 10:10:00
categories: PHP
tags: PHP
---
做互联网系统 分词是必不可少的。除非咱不需要搜索、标签或者关键字分析等。

在网上看到的一个列表，不错，保存以备不时之需

BosonNLP：<http://bosonnlp.com/dev/center>
IKAnalyzer：<http://www.oschina.net/p/ikanalyzer>
NLPIR：<http://ictclas.nlpir.org/docs>
SCWS中文分词：<http://www.xunsearch.com/scws/docs.php>
结巴分词：<https://github.com/fxsjy/jieba>
盘古分词：<http://pangusegment.codeplex.com/>
庖丁解牛：<https://code.google.com/p/paoding/>
搜狗分词：<http://www.sogou.com/labs/webservice/>
腾讯文智：
<http://www.qcloud.com/wiki/API%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3>

 如果是为了快速实现功能，而且我们没有太多的二次开发力量，那么我们应该使用REST API 模式的分词接口。直接调用，人家更新我们也自动跟着更新。不过，一旦人家歇菜，我们也跟着歇菜。所以使用REST API风格的分词要做好后手准备。另外，如果你需要有一些个性化功能，是木有的，这就是 “懒”和”笨“的代价。

 REST API接口的大家可以试一下。BosonNLP和新浪云，如果专注中文，那么新浪是比较好的选择。新浪早年的产品,譬如上个世纪90年代，还是很差的。现在的新浪很多产品还是值得学习的。(呀~~~不小心暴露了年龄)

**PHP分词**

如果你认为PHP是世界上最好的语言，那么选择SCWS是必须的啦。需要安装扩展，自己可以修改词库，配置也方便。适合于PHP大法传人和有一定二次开发能力的人。如果你能修改源码那就更屌了。