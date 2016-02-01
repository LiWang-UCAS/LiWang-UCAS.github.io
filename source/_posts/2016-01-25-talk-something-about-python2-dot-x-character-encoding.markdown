---
layout: post
title: "Python2.x 字符编码的一些问题"
date: 2016-01-30 11:44:58 +0800
comments: true
categories: Python 
---
本文主要包括以下内容：

> * 略谈字符编码
> * Python2.x 字符编码的那些事

<!--more-->

### 字符编码

既然讲 Python2.x 字符编码的一些问题，必须先简单介绍下字符编码。关于字符编码的详细内容可以参考 [wiki](https://en.wikipedia.org/wiki/Character_encoding)。

这里说说我的理解：字符编码作为名词表示的一些符号的集合；作为动词是将字符进行编码，是将一组符号（集合）映射到另外一组符号（集合）。字符编码的目的可以是为了存储，计算或者传输。

随着计算机的出现，自然语言中的符号只有通过编码后才能在计算机中存储，计算或者传输。因为计算机只认识二进制，所以必须将自然语言中的字符映射为二进制序列。这样，ASCII 编码系统就诞生了。

ASCII 编码系统包括字符集、编码表及存储方式。ASCII 是单字节编码，其字符集大小为256，分别是0-255之间的整数（十进制表示），对应着自然语言中的256个符号，这种对应关系构成了 ASCII 的编码表。但如何将 ASCII 字符集的256个整数映射到二进制序列集合上呢。最简单的办法就是直接将十进制整数进行二进制编码。其实 ASCII 编码系统也是这么干的。但 ASCII 码只能表示256个字符，对于汉语这样有几万个字，显然 ASCII 码的字符集太少了。因此，必须扩展 ASCII。为了扩充 ASCII 编码，以用于显示本国的语言，不同的国家和地区制定了不同的标准，由此产生了GB2312、BIG5、 JIS 等各自的编码标准。这些编码系统的字符集中有65536个字符（可以简单理解为0-65535之间的整数），使用2个字节存储，统称为 ANSI 编码。

不知道大家看到这里是否发现一个问题：不同的编码系统中的编码表规定的对应关系可能不一样，这样会带来很多麻烦。如：在 ANSI 编码下，同一个编码值（这里可以认为是0-65535之间的一个整数），在不同的编码体系里代表着不同的符号。在简体中文系统下，ANSI 编码代表 GB2312 编码，在日文操作系统下，ANSI 编码代表 JIS 编码，可能最终显示的是中文，也可能显示的是日文。在 ANSI 编码体系下，要想打开一个文本文件，不但要知道它的编码方式，还要安装有对应编码表，否则就可能无法读取或出现乱码。既然会有这么多麻烦，为什么不制定一个编码系统，将世界上所有的符号都纳入其中，无论是英文、日文、还是中文等，大家都使用这个编码系统呢。Unicode 编码系统因此而生。

Unicode 编码系统同样包含字符集、编码表和字符集存储方式（实现方式）。关于 unicode 的详细资料可以参考[wiki](https://en.wikipedia.org/wiki/Unicode)。Unicode 码的实现方式有很多,如：utf-8，utf-16。常用的是：utf-8。utf-8规定了unicode 中的字符和二进制序列的对应关系。下面这张图应该很好的解释 utf-8 和 unicode 的关系。

![unicode与utf-8关系图](http://7xqgif.com1.z0.glb.clouddn.com/unicode-utf8.png?imageView2/2/w/500)

PS: 这里抛出两个问题：第一：为什么不直接将 unicode 编码字符的二进制作为存储编码呢？第二：为什么 utf-8 的编码最大长度为6个字节而不是4个字节？

### Python2.x 字符编码的那些事

想必大家在用 Python 处理中文时，难免会碰到乱码、SyntaxError 或者 UnicodeEncodeError。这时候您也许会到网上搜索解决方案，但很多答案都只说了解决方案，很少深究原因。下面讲讲我对 Python 中字符编码问题的一些理解和总结。

* Python 脚本文件中含有非 ASCII 字符（如：中文字符）

{% codeblock lang:python %}
testStr="汉"
print testStr
{% endcodeblock %} 

上面的 Python 代码，如果以脚本文件 test1.py 保存并运行，会出现:

> SyntaxError: Non-ASCII character ' \xe6 ' in file test1.py on line 1, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details

上面错误信息：符号错误，文件 test1.py 中的第一行出现非 ASCII 字符 ' \xe6 '，但没有字符编码的声明。

其实错误提示已经很清楚了。之所以出现这样的错误是因为在作者的电脑上设定保存文件的编码是：utf-8（“ 怎么查看当前系统上保存文件的编码 ” 这个问题下面会讲），因此中文字符 “ 汉 ” 存储为 “ \xe6\xb1\x89 ”（十六进制表示）。Python 解释器默认以 ASCII 解码（此处 ASCII 码的字符集范围为: 00000000 ~ 01111111），在解析 test1.py 文件时，读到 “ 汉 ” 的编码的第一个字节 “ \xe6 ”（二进制为：11100110）时，发现 11100110 不在 00000000 ~ 01111111 之内。因此出现了以上的错误信息。

至于怎么解决这个问题，大家如果想了解更清楚，可以查看 [http://python.org/dev/peps/pep-0263/](http://python.org/dev/peps/pep-0263/)。简单来说：在文件的第一行进行字符编码的声明就 OK 了：如下：

{% codeblock lang:python %}
# -*- coding: utf-8 -*-
testStr="汉"
print testStr
{% endcodeblock %}

* Python 中的解码（decode）与编码（encode）

Python 中有两个字符串类，分别为：str 和 unicode。str 和 unicode 都是 basestring 的子类。严格意义上说 str 是字节串。str 类的实例保存的内容为字符串按照指定编码而编码后的字节串。而 unicode 类的实例保存字符串按照 unicode 编码而编码后的内容。请看下面 Python 代码：

{% codeblock lang:python %}                                                     
# -*- coding: utf-8 -*-                                                
testStr="汉"
testUnicode=u"汉"
print "testStr is " + repr(testStr)
print "testUnicode is "+repr(testUnicode)
{% endcodeblock %}

运行结果为:

{% codeblock lang:python %}
testStr is '\xe6\xb1\x89'
testUnicode is u'\u6c49'
{% endcodeblock %}

运行结果正如上面所示：testStr 的内容为 “汉” 的 utf-8 编码十六进制表示，testUnicode 的内容为 “汉” 的 unicode 编码十六进制表示。

Python 中的编码与解码是相对于 unicode 编码而言的。即：其他编码到 unicode 编码转换的过程称为解码，unicode 编码到其他编码的过程为编码。一图以蔽之。

![encode and decode](http://7xqgif.com1.z0.glb.clouddn.com/encode_decode.png?imageView2/2/w/500)

下面是关于str解码和unicode编码的示例代码：

{% codeblock lang:python %}
# -*- coding: utf-8 -*-                                                
testStr1="汉"
testUnicode1=testStr1.decode("utf-8")
testUnicode2=unicode(testStr1,"utf-8")
testStr2=testUnicode1.encode("utf-8")
print "testStr1 is " + repr(testStr1)
print "testStr2 is " + repr(testStr2)
print "testUnicode1 is "+repr(testUnicode1)
print "testUnicode2 is "+repr(testUnicode2)
{% endcodeblock %}

上面代码运行的结果为：

{% codeblock lang:python %}
testStr1 is '\xe6\xb1\x89'
testStr2 is '\xe6\xb1\x89'
testUnicode1 is u'\u6c49'
testUnicode2 is u'\u6c49'
{% endcodeblock %}

从代码和运行结果可以知道，字节串可以通过调用 decode() 或者 unicode() 函数，将字节串解码为 unicode 字符串。Unicode 字符串通过调用 encode() 函数将 unicode 字符串编码为指定编码的字节串。

那字节串是否可以调用 encode() 函数呢。按照道理，应该不行。但 Python 中允许字节串调用 encode() 函数，那是为什么呢？其实字节串调用 encode() 函数会发生两个过程：首先字节串会按默认编码解码为 unicode 字符串，然后将 unicode 字符串编码（encode）为指定字节串。因此这里有一个需要注意的地方：如果系统默认编码不是字节串的编码方式，那就会出现错误。请看下面代码：

{% codeblock lang:python %}
# -*- coding: utf-8 -*-
testStr1="汉"
testStr2=testStr1.encode('gbk')
print repr(testStr2)
{% endcodeblock %}

运行上述代码，会出现一下错误：

```
Traceback (most recent call last):
  File "xxx.py", line 3, in <module>
    testStr2=testStr1.encode('gbk')
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe6 in position 0: ordinal not in range(128)
```
至于为什么会出现这样的错误呢？我只能告诉你：那是因为系统默认编码为 “ascii”，。。。。此处的省略号是故意留下的，因为如果你还不知道怎么分析，那你还是重新仔细读读我上面所说的内容吧。

对与上面出现的错误，我们只需要设定（Python）系统默认编码为 “utf-8” 即可，系统默认编码的缺省值为 “ascii”。那么问题又来了，该怎么设置定系统默认编码呢？请看下面代码：

{% codeblock lang:python %}
# -*- coding: utf-8 -*-
import sys
reload(sys)
sys.setdefaultencoding("utf-8") #设定系统默认编码为 “utf-8”

testStr1="汉"          
testStr2=testStr1.encode('gbk')
print repr(testStr2)
{% endcodeblock %}

大家有没有注意到，上面的代码为了调用 sys.setdefaultencoding("utf-8")，sys 模块导入了一次，重新加载了一次。这是为什么呢？在这里作简单解释：在 Python 解释器启动时，会自动加载 [site.py](https://docs.python.org/2/library/site.html) 模块，大家可以看看这个模块的代码，其中 在 main() 函数中有这么几行代码:

{% codeblock lang:python %}
# Remove sys.setdefaultencoding() so that users cannot change the
# encoding after initialization.  The test for presence is needed when
# this module is run as a script, because this code is executed twice.
if hasattr(sys, "setdefaultencoding"):
   del sys.setdefaultencoding
{% endcodeblock %}

由以上代码可知，sys 中的 setdefaultencoding() 函数在初始化后就被删除了。如果用户需要设定系统默认编码，必须首先调用 reload(sys)，然后调用 sys.setdefaultencoding() 就 OK 。

讲完 Python 中字节串 encode 的原理后，补充一下：其实在 Python 中 unicode 字符串也是允许调用 decode() 函数，但意义不大（因为如果运行结果正确，得到的结果还是本身的 unicode 字符串）。其中的原理和字节串 encode 相似。Unicode 字符串调用 decode() 函数也会发生两个过程：首先 unicode 字符串按照系统默认编码而编码为字节串，然后字节串按照指定编码解码为 unicode 编码。如果系统默认编码和调用 decode() 指定的编码不同，就会出现乱码或错误。

本篇博文到这里快接近尾声了，下面粘贴下有关于 Python 中与编码相关的方法。

{% codeblock lang:python %}
import sys
import locale  

print sys.getdefaultencoding() #获取当前Python中使用的默认字符编码
print locale.getdefaultlocale() #获取当前操作系统中默认的区域设置并返回元祖(语言, 编码)
print locale.getpreferredencoding() #获取用户设定的文本数据编码
{% endcodeblock %}

关于 Python 中的 locale 模块，大家可以参考：[https://docs.python.org/2/library/locale.html](https://docs.python.org/2/library/locale.html)

