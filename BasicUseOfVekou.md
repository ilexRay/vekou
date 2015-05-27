### 本篇文章，我将会手把手教你建立一个可以立即运行的Vekou环境。 ###


1. 首先，您的电脑必须有一个Java基础环境，我在这里不多说。

2. 从<a href='http://code.google.com/p/vekou/'><a href='http://code.google.com/p/vekou/'>http://code.google.com/p/vekou/</a></a>中下载<a href='http://vekou.googlecode.com/files/dict.7z'>dict.7z</a>，<a href='http://vekou.googlecode.com/files/properties.7z'>properties.7z</a>，<a href='http://vekou.googlecode.com/files/Vekou.jar'>Vekou.jar</a>，<a href='http://vekou.googlecode.com/files/lib.7z'>lib.7z</a>，<a href='http://vekou.googlecode.com/files/jyutping-wong-44100-v7.7z'>jyutping-wong-44100-v7.7z</a> 这5个文件，其中最后一个文件差不多100M，是voicelib。

3. 如果你有exlipse，那就建立一个Java project，将工程的text encoding改成UTF-8，新建一个Test.java，复制<a href='http://www.webyouknow.com/?p=255'>Vekou，粤语的语音合成</a>中的代码到这个文件中。

4. 在工程文件夹下建立一个lib文件夹(其实你随便起一个什么名字都可以)，把Vekou.jar以及从lib.7z中解压出来的所有jar文件都放在其中，在工程的Java Build Path中的Libraries中把所有包都添加上。

5. 在工程文件夹下建立data和properties文件夹，把properties.7z解压出来的所有文件放到properties文件夹中，，把jyutping-wong-44100-v7.7z和dict.7z解压出来的文件放到data文件夹中。

6. 运行Test.java尝试一下！

Vekou现在所有的使用方法都在Test.java中展示了，当然你可以自定义一个ProcessCenter来处理出更和谐的声音出来，你也可以在各种字典中添加词汇来增强Vekou的识别能力，甚至你可以从别的地方找来一个jar包使Vekou可以读出adobe reader或者word甚至html中的文字，只要这个包可以把文件中的文字转换成String就没问题。这些问题我将会包括在<a href='http://www.webyouknow.com/?p=288'>Vekou 高级</a>使用中讲到。