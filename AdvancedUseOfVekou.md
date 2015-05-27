没有实现time scaling与pitch scaling是一个遗憾，特别是pitch scaling比较难实现，如果学了数字信号处理那应该是很easy的事，听说是要用到DFT的，准确来说是频域上移频，我借了几本书回来，等我刨完那些书以后在写应该就没问题了。time scaling主要是现在没时间写，原理是在音频流中插帧，在原本的两个帧之间插入一个左右帧的平均帧，递归插，需要延长多少时间就插多少，这个功能我一有时间就一定去实现。

怎么说，没有了这两个功能语音合成就好像成了鸡肋，当然最主要的还是如何处理初始音频流，使它的发音尽量接近人类的声音。

本篇文章会从Vekou的结构说起，到基本的配置，到你应该如何自定义一个音频流处理器，统统都会说到的。

说<strong>结构</strong>，人人都喜欢用图表示。
<br />
<a href='http://www.webyouknow.com/wp-content/uploads/2010/02/1.jpg'><img src='http://www.webyouknow.com/wp-content/uploads/2010/02/1.jpg' alt='' height='189' width='368' title='1' />
</a><br />
首先，从外表看，利用Vekou就相当于产生了一个发动机(Engine class)，它要启动，需要一个处理器的，处理器教它怎样启动。

那么ProcessCenter具体处理什么呢？<br />
<a href='http://www.webyouknow.com/wp-content/uploads/2010/02/2.jpg'><img src='http://www.webyouknow.com/wp-content/uploads/2010/02/2.jpg' alt='' height='189' width='553' title='2' /></a><br />
ProcessCenter主要运用两大部件，第一个是IndexManager，第二个是AnalyzeCenter。当Vekou从外部获取到文本以后，首先交给AnalyzeCenter分析，把哪些该停顿的地方，哪些是一个词组(中文分词)用一个LinkedList保存住，然后交给IndexManager处理。IndexManager得到数据以后，当遇到是文字的地方，如果需要进行粤语口语转换，就在口语字典中找到对应的口语表达，然后找到字所对应的语音文件，当遇到是标点的时候，在字典中找到标点所对应的停顿时长权值，把这些所有的信息都存储到一个ArrayList中。

之所以在分析阶段选择存储到LinkedList中而不是ArrayList中是因为考虑到在口语转换过程中可能要进行频繁的元素替换，甚至有时候可能是多个元素替换成一个元素，所以选择用LinkedList进行这样的操作是比较快速的。

当ProcessCenter得到分析及索引以后的数据后，就开始整合出一个音频流出来<br />
<a href='http://www.webyouknow.com/wp-content/uploads/2010/02/3.jpg'><img src='http://www.webyouknow.com/wp-content/uploads/2010/02/3.jpg' alt='' height='189' width='472' title='3' />
</a><br />上面的图片其实就表达了DefaultStreamProcess(implements ProcessCenter)是如何处理音频流的，当遇到一个词组的时候，比如遇到双字词，第一个字就切去右边的数帧，第二个字就切去左边的数帧，如果遇到一个三字词，第一个字切去右边数帧，第二个字左右均切去数帧，第三个字切去左边数帧，如此类推。遇到一个字的时候，默认切去左边数帧。

上面已经说了，当遇到标点符号的时候，IndexManager就做如下处理<br />
<a href='http://www.webyouknow.com/wp-content/uploads/2010/02/4.jpg'><img src='http://www.webyouknow.com/wp-content/uploads/2010/02/4.jpg' alt='' height='189' width='474' title='4' />
</a><br />所以在DefaultStreamProcess中就可以根据这些权值做插入一些空白(无声)的音频流<br />

<a href='http://www.webyouknow.com/wp-content/uploads/2010/02/5.jpg'><img src='http://www.webyouknow.com/wp-content/uploads/2010/02/5.jpg' alt='' height='189' width='375' title='5' /></a><br />
最后，SpeechEngine(implements Engine)运用SimpleAudioPlayer发出最后的声音。

Vekou整个处理步骤就说完了。

<strong>配置</strong>文件：

在Vekou 基础使用中提到需要建立一个properties和data文件夹，其实这是在properties中的config.properties文件中定义的，我默认就定义成这样，你可以用写字板打开config.properties文件，如果你没有改动到它，应该看到如下内容：
<div>index.txtDict.dir=/data/zhy.dict</div>
<div>index.swDict.dir=/data/stopwords.dict</div>
<div>index.pron.dir=/data/jyutping-wong-44100-v7</div>
<div>index.oralDict.dir=/data/oral.dict</div>
以上所有“/”表示你存放Vekou.jar包所在文件夹的上层文件夹，比如包在D:/project/speech synthesis/Vekou.jar位置，那么“/”表示的位置就是D:/project/，这就是说如果你自定义一个存放properties文件和data文件的位置的时候，必须要以“/”开始，否则程序会报找不到路径的错误。之所以这样做，因为考虑到工程一般设置一个专门的文件夹来存放jar文件，而这个文件夹一般和src文件夹是同级的，这么设计会美观一些。

回到config.properties文件定义上来，第一行定义粤语词组字典文件的位置，第二行定义停顿符号文件的存放位置，第三行定义语音库存放位置，第四行定义书面语转口语映射关系字典的存放位置。你可以自己往任意一个词典增加词组或者在音库中增加发音文件，如何增加、有什么规则我将会在后面谈到。

<strong>扩展</strong>Vekou：

一、ProcessCenter
ProcessCenter是一个接口，其中定义了2个静态常量和9个方法头：
<ol>
<blockquote><li>静态常量：<br>
PUNCTUATION_WAIT_TIME，它定义了单位权值停顿的毫秒数，权值×PUNCTUATION_WAIT_TIME=该标点符号停顿的毫秒数<br>
EXTERNAL_BUFFER_SIZE，它定义了在音频流处理过程中所用到的byte数组的长度，一般情况下不需要管这个常量。</li>
<li>方法头<br>
public AudioInputStream pronounceElementProcess(List&lt;Object<a href='.md'>.md</a>&gt; list) throws SpeechSynthesisException; 从文本的词组，停顿符信息中构造出一段音频流<br>
public AnalyzeCenter getAnalCen(); 获得分析器<br>
public void setAnalCen(AnalyzeCenter analCen); 设置分析器<br>
public IndexManager getIndexManager(); 获得索引器<br>
public void setIndexManager(IndexManager indexManager); 设置索引器<br>
public float getPitch(AudioInputStream stream); 从音频流中获得音调(没有实际操作，暂时不要使用)<br>
public void setPitch(float pitch); 设置音调(暂时不要使用)<br>
public long getDuration(AudioInputStream stream) throws NumberFormatException, UnsupportedAudioFileException,IOException; 从音频流中获得总播放时长(以毫秒计算,没有实际操作,暂时不要使用)<br>
public void setDuration(long duration); 设置音频播放时长(暂时不要使用)</li>
</ol>
当你自定义一个更好的更复杂的流处理器的时候需要implements ProcessCenter，你就可以不用忍受这么机械化的语音声音了。</blockquote>

二、AnalyzeCenter
当你不喜欢imdict-chinese-analyzer分词器的时候，你可以重新设计一个分析器，AnalyzeCenter也是一个接口，它有两个方法头：
<ul>
<blockquote><li>public List&lt;String&gt; doSegment(String str) throws IOException; 用于把文本分词，切割成一个一个词组，然后存到List中</li>
<li>public List&lt;String&gt; convert(String str); 用于把一个词组或者一个单字转换成unicode编码表示，Vekou的字典全部都是用unicode码进行索引</li>
</ul>
三、FileScanner<br>
Vekou默认只可以从txt文档中读取文本，你也可以从别的地方获得一个包，只要可以从一个文件中抽取到文本到String中，就可以让Vekou朗读这个文件的文字。FileScanner是一个接口，它只声明了一个方法头：<br>
<ul>
<li>public String conver(File file) throws IOException; 从一个文件中读取得到String返回</li>
</ul>
实现了这个接口以后，你必须为这个类负责的一类文件注册。如何注册？在fileTypeRegistry.properties文件中已经为txt文件注册了org.lib.speech.util.TxtFileToString读取器，参照这种方法为你的读取器注册，然后Vekou就可以读取这样的文件了。</blockquote>

自定义/修改<strong>字典</strong>：

所有的字典文件都支持以“//”开头的注释，每个字典开头的注释都已经写明了如何插入新词汇，你可以打开字典文件查看。例如如果你发现有一些词组中的某个字Vekou把它读错了，你可以在zhy.dict中增加这整个词组的读音定义。可以想象，字典越丰富，Vekou的发音越准确。又或者你尝试让Vekou使用粤语口语发音，但是有些词仍然用书面语形式读出来，那么你可以在oral.dict中定义这个词的映射关系，下载Vekou再读到这个词的时候它就知道该如何读了。其实Vekou是很笨的，你教它什么它才会什么，不教的话什么都不会。

<strong>附注：</strong>
<ol>
<blockquote><li>Vekou运行的时候会生成或者读取(在ProcessCenter的boolean参数中设置)字典的序列化文件，默认保存在jar所在目录的上层目录(也就是“/”)的data文件夹中，以加快运行速度；</li>
<li>Vekou使用文件使用UTF-8编码，这是为了简繁都支持。Vekou的SpeechEngine中有繁简转换器，当文本是繁体的时候，内部是转换成简单再作处理的，这个功能是靠ZHConverter.jar包实现；</li>
<li>Vekou读取txt文本可以自动检测文档文件的编码，使你不用担心乱码问题。这个功能是靠antlr.jar、chardet.jar、cpdetector_1.0.7.jar三个包实现。</li>
<li>中文分词是靠chinese-analyzer.jar(已修改版)、lucene-core-2.9.1.jar两个包实现</li>
</ol>
另外，还有一个<strong>非常非常重要</strong>的问题，是什么问题...我将会在<a href='http://www.webyouknow.com/?p=303'>下一篇文章</a>中谈到。