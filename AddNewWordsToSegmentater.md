在上一篇文章中提到还有一个很重要的问题没有说清楚，其实就是关于Vekou底层所涉及到的中文分词器imdict-chinese-analyzer，如果理解了Vekou 高级使用中谈到Vekou内部是如何工作的以后，就知道如果Vekou遇到一个从来未见过的新词，就算向zhy.dict添加一个这个新词，如果在分词阶段没有把这个词正确划分，Vekou就不能像一个词一样把它读出啦，听上去依然像一个字一个字地读。所以，要让Vekou正确识别一个词，有两个条件必须具备：
<ol>
<blockquote><li>必须让中文分词器划分出这个词，即分词器字典中存在这个词</li>
<li>Vekou中的粤语发音字典中(zhy.dict)也要存在这个词</li>
</ol>
之所以这样做，因为考虑到Vekou不是专注于中文分词，这只是其中一个使用到的模块，别人也许可以不断开发出更好的中文分词器，这样Vekou就可以站在巨人的肩膀上提高性能。</blockquote>

那么，如何向imdict-chinese-analyzer分词器增添新词呢？

这个中文分词器在<a href='http://www.ictclas.org'>ICTCLAS.org</a>官网可以<a href='http://www.ictclas.org/OpenSrcDownCount.asp?PacketId=48&amp;url=down/imdict-chinese-analyzer.zip'>下载</a>，不过Java版本的没有相应向字典里加词的方法，但C#版里面有，然后运行SharpICTCLAS工程里的AddWords2Dict，可以按里面的运行代码增加一个新词，不过要注意，只限于增加简体词汇，不支持繁体，不过这个问题你不用担心，Vekou本身已经支持繁体了，实际上Vekou内部都是转换成简体进行操作的，只要你在分词字典中增加对应的简体词汇，无论繁体简体，Vekou都能接受！

向imdict-chinese-analyzer字典中增加了一个新词以后，AddWords2Dict默认会重新生成一个包含原数据和新词汇的新词典，如果你要使这个字典能够让Vekou读取，需要重新命名为coredict.dct。

但是！！

你下载的chinese-analyzer包中是没有这个字典的，它只有这个词典的序列化文件！就是在\net\imdict\wordsegment\dictionary\下的coredict.mem文件。所以你必须把刚刚产生的包含新词的新词典转换成这个样子的东西才行。haha！这个你就不用担心了，只要你下载<a href='http://vekou.googlecode.com/files/chinese-analyzer.7z'>这个Java Project</a>，把\analysis-data\coredict.dct替换掉，删除已经生成的旧coredict.mem文件，然后运行\test\net\imdict\analysis\testAnalyzerTest.java，当你再次打开\analysis-data\的时候，你就会看到你想要的文件了，注意！这个文件和之前你删除的coredict.mem不同，这次这个已经包含了你刚才插入的新词汇。接下来的工作当然是把chinese-analyzer.jar重新打包就可以了使用了。

需要注意的是，你下载的<a href='http://vekou.googlecode.com/files/chinese-analyzer.7z'>chinese-analyzer.7z</a>和在官网上的有一点点不同，官网上的那个在重新生成序列化文件的时候会报错，原因是原程序的bug，有一个地方返回空指针却没有抛出错误，我已经修正了。

Vekou是一个开源程序，它很需要你的哪怕一点智慧或者力量来强化身体，我非常欢迎你随时联系<a href='mailto:ljishen@gmail.com'>我</a>。