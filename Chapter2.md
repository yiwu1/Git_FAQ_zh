### 2 特性 ###
#### 2.1 Git会在不同平台之间自动转换CRLF和LF吗？####
#### 2.2 Git支持关键字替换吗？####
#### 2.3 Git支持任意的内容转换吗？####
#### 2.4 Git会转换文件名的字符编码吗？####
#### 2.5 Git会转换提交说明，文件内容的字符编码吗？####
#### 2.6 Git会track关于文件的所有数据和元信息吗？####

### 特性 ###

### Git会在不同平台之间自动转换CRLF和LF吗？###
__________________
Git从版本1.5开始已经支持EOL转换，请参考[gitattributes](http://www.kernel.org/pub/software/scm/git/docs/gitattributes.html#_checking_out_and_checking_in)

### Git支持关键字替换吗？###
__________________
Git并不推荐使用关键字替换。关键字替换会引起一系列奇怪的问题而且它也没有你想象的那么有用——尤其在SCM的世界中。你可以通过脚本在Git之外实现关键字替换。例如，Linux kernel的export脚本通过在Makefile中设置变量EXTRA_VERSION来实现。  
参考[gitattributes](http://www.kernel.org/pub/software/scm/git/docs/gitattributes.html)，如果你真的要这么做。如果你的转换是不可逆的，那么很有可能会出问题。(比如: $Id$会被替换成"$Id:"后跟40字节的blob id后跟"$"）   

- 点此查看[关于该问题的讨论](http://thread.gmane.org/gmane.comp.version-control.git/44750)以及[另一个讨论](http://article.gmane.org/gmane.comp.version-control.git/44849)

### Git支持任意的内容转换吗？###
__________________
是的。 除了"关键字替换"和"CRLF转换"以外，当前的Git版本允许你在提交文件到Git仓库之前*munge contents*。详细信息请参考[gitattributes](http://www.kernel.org/pub/software/scm/git/docs/gitattributes.html)

### Git会转换文件名的字符编码吗？###
__________________
不会，文件名只会被视为字节流。

### Git会转换提交说明，文件内容的字符编码吗？###
__________________
Git仓库可以通过一个选项来设置本仓库中提交信息(包括提交者信息)所使用的编码格式。文件内容的字符编码不会被转换除非你想搬起石头砸自己的脚。如果你真想那么做的话，参考前面提到的过滤机制。

### Git会track关于文件的所有数据和元信息吗？###
__________________
不会。 Git对“track的数据”这一概念有着自己的解读，通常情况下仅仅指文件内容。Git通常并不适用于文件内容依赖于额外的文件系统信息的情况(*注:比如/etc目录，来自该目录的文件通常是一些配置文件，但是对于Git来说它们只是一些普通的文本文件，而不会记录它们来自/etc目录*)。参考[内容限制](https://git.wiki.kernel.org/index.php/ContentLimitations)来获取更多相关信息。