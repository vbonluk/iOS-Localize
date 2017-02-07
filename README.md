
参考链接：

<http://devma.cn/blog/2016/05/27/ios-duo-yu-yan-ben-di-hua/>

<http://blog.lessfun.com/blog/2016/12/26/ios-app-strings-translator/>

### 场景：

	为已经完整开发的大型项目添加多语言的支持。
	由于开发初期的开发者完全没有考虑到APP会支持多语言的情况
	没有使用Localizable.strings和NSLocalizedString(key, comment)
	
现在忽然要开始支持多语言了，在开发了很长一段时间后，程序所积累的代码估计得上10万行。没有使用Localizable.strings和NSLocalizedString(key, comment)，那么你就需要一步一步去找到每个地方去添加支持。想想就可以知道这个重复过程有多痛苦。

下面结合我自己的实际情况，给大家提供了一些简便的脚本工具，可以节约一半时间吧。

注意：以下的工具都是Python脚本，注意配置你电脑的Python环境。

### 工具一：

AppStringsTranslator

### 原理

	扫描 App 的 xx.strings 文件，遍历每一行。
	如果该行格式为 "xx" = "xxx";，则把等号左右的key 和 value提取出来，保存到 keys 和 values 两个数组中。
	如果该行不是 key/value 格式，则把整行加到 keys，而往 values 里加一个空字符串。
	利用百度翻译的 API，翻译 values 里的每个元素。
	把 keys 和翻译结果按照同样的格式写入到新的 xx_toLang.strings 文件。
	第 3 步的目的是，保留原来的语言文件里的注释和空行。

### 用法

	修改 AppStringsTranslator.py，填上你自己的百度 AppKey/SecretKey，见这里。
	在终端执行脚本，eg: python AppStringsTranslator.py。
	输入语言文件名称，eg: Localizable.strings。
	输入源文件的语言简写，eg: zh。
	输入需要翻译成的语言列表，以空格区分。eg: en cht jp kor。
	等待完成。
	
![](http://oapglm9vz.bkt.clouddn.com/1486438132.png )

### 支持的语言

![](http://oapglm9vz.bkt.clouddn.com/1486438170.png )

### 无法脚本化的情况

有时候，我们会在语言文件里添加一些特殊的字符，比如 %@ %d %lld 之类的格式化占位符，这些字符是不需要被翻译的。但是利用脚本，是没有办法智能识别哪些字符需要保留，以及保留到哪里的，这种情况还是需要开发者自己去处理。

注意事项：
	
	注意自动翻译后的结果：在简体翻译成繁体的时候，
	有时候百度翻译会出问题，将“搜索”翻译成“蒐索”，
	这是错误的。需要手动修改成正确的“搜索”；

### 工具二： AutoGenStrings.py

这个工具是用于xib和storyboard的。

在xib和storyboard中支持多语言并不需要用到NSLocalizedString(key, comment)。

![](http://oapglm9vz.bkt.clouddn.com/1486438637.png )

![](http://oapglm9vz.bkt.clouddn.com/1486438664.png )

上面两张图片可以知道Localizable.strings存在于xib或者storyboard中。我们可以进去编辑文字即可。但是这里往往会出现一个问题，即xcode只会对刚将xib或者storyboard设置成支持多语言的时候界面上存在的UI控件的描述添加进Localizable.strings。也就是说，你后面在这个xib或者storyboard新增一个UI控件，xcode并不会帮你将其添加进去Localizable.strings。不过还好Xcode为我们提供了 ibtool 工具来生成XIB的strings文件来帮助我们自动添加进去：

终端命令：
		
	ibtool Main.storyboard --generate-strings-file ./NewTemp.string

但是ibtool生成的strings文件是BaseStoryboard的strings(默认语言的strings)，且会把我们原来的strings替换掉。所以我们要做的就是把新生成的strings与旧的strings进行冲突处理(新的附加上，删除掉的注释掉)。

这一切可以用这个pythoy脚本来实现（AutoGenStrings.py）

然后我们将借助 Xcode 中 Run Script 来运行这段脚本。这样每次Build时都会保证语言strings与Base XIB保持一致

![](http://oapglm9vz.bkt.clouddn.com/1486439002.png )

		
	python ${SRCROOT}/RunScript/AutoGenStrings.py ${SRCROOT}/${TARGET_NAME} //参数为StoryBoard，XIB所在目录

上面的文件我放在了这里：<https://github.com/vbonluk/iOS-Localize>

原创作品，欢迎转载，转载请声明出处：<http://www.真无聊.com>
 
友情链接联系:5914018@qq.com
 
交流QQ群：271568188
