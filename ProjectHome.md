至于为什么叫Vekou，我也不知道，唯一想了一分钟多的是用什么字母开头，i是苹果，z和m、w都是微软，k是相机和胶卷，d像狗，所以还是v比较有型一点，然后随手就把后面几个可以连在一起发音的字母敲出来了。Vekou读作['vekau]，项目进行了一段时间以后才突然想起需要一个比较像样的名，一开始建eclipse项目名用的还是speech synthesis。

Vekou目前虽然发音质量不是相当好，但基本上还可以工作了。你可以给一个String让它发音，也可以给一个txt文档让它发音，当然你也可以让程序给你生成一个语音文件。0.0.4版的功能详细的功能说明如下：
<ol>
<blockquote><li>String发音</li>
<li>txt文档文件发音，txt文档编码自动检测</li>
<li>语音文件生成</li>
<li>粤语口语转换发音</li>
<li>time scaling(未实现)</li>
<li>pitch scaling(未实现)</li>
<li>繁简支持</li>
<li>语音库自定义</li>
<li>词典自定义</li>
<li>口语字典自定义</li>
</ol>
Vekou的诞生离不开<a href='http://e-guidedog.sourceforge.net/ekho_cn.php'>Ekho(余音)</a>的支持，Vekou的语音库以及初始词典全部来自于它，还有基于中科院的<a href='http://code.google.com/p/imdict-chinese-analyzer/'>imdict智能词典所采用的智能中文分词程序</a>，Vekou的底层使用到中文分词。</blockquote>

你可以很简单的使用它，初次尝试的时候你可以建立一个如下的Test.java文件来测试：

```
import java.io.File;
import java.io.IOException;
import java.util.Iterator;
import java.util.List;

import org.lib.speech.engine.Engine;
import org.lib.speech.engine.SpeechEngine;
import org.lib.speech.process.DefaultStreamProcess;
import org.lib.speech.process.ProcessCenter;

public class Test {
	public static void main(String[] args) {

		// 建造一个流处理器，参数设置是否重新读取字典文件
		ProcessCenter pc = new DefaultStreamProcess(true);

		// 建立一个语音引擎，第二个参数设置是否转换为粤语口语发音
		Engine engine = new SpeechEngine(pc, true);

		// 任何一个String作为你想要它发音的句子
		String sentences = "你可以在这里尝试任何一个句子，看看它是如何发音的。";

		// 第一种方法：直接要它发音
		engine.getPronounces(sentences);

		// 第二种方法：句子在一个txt文档中，你要它把txt中的内容读出来，第二个参数设置是否将文档内容输出到控制台显示
		try {
			engine.getPronounces(new File("C:/a.txt"), false);
		} catch (IOException e) {
			e.printStackTrace();
		}

		// 第三种方法：把发音保存在一个.au的声音文件中，目前只支持保存到这种文件，当然你也可以自己扩展
		try {
			engine.getPronouncesFile(sentences, new File(
					"C:/a.au"));
		} catch (IOException e) {
			e.printStackTrace();
		}

		// 另外，如果你想获得初始的发音素材，可以这样显示到控制台
		List<Object[]> list = engine.getPronounceElements(sentences);
		Iterator<Object[]> iter = list.iterator();
		while (iter.hasNext()) {
			Object[] obj = iter.next();
			if (obj[0] instanceof File) {
				for (int i = 0; i < obj.length; i++) {
					File file = (File) obj[i];
					System.out.print(file.getName() + " ");
				}
			} else {
				for (int i = 0; i < obj.length; i++) {
					System.out.print(obj[i] + " ");
				}
			}
			System.out.println();
		}
	}
}
```