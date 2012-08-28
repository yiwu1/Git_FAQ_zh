#1 常见问题#
####1.1 什么是git？####
####1.2 为什么取名git？####
####1.3 如何报告git的bug？####
####1.4 fetch和pull有什么差别？####
####1.5 我可以使用哪些资源来建立一个公共仓库？####
####1.6 我可以添加空目录吗？####
####1.7 为什么git不记录rename操作？####
####1.8 为什么git log <filename>操作会慢？####
####1.9 为什么export环境变量CDPATH是错误的？####
####1.10 为什么gitweb突然从kernel.org/git上消失了？####
####1.11 merge和rebase有什么差别？####
####1.12 为什么git rm不是git add的逆操作？####


### 什么是git？###
_____________________
Git是一个由Junio Hamano和Linus Torvalds开发的分布式版本控制系统。  
Git不需要一个集中的服务器。   
Git可以运行在Linux, BSD, Solaris, Darwin, Windows Android等操作系统上。              

### 为什么取名git？###
_____________________
引用Linus的一句话:
> 我是一个自私的\*\*\*，我的所有项目都以我命名。先是‘Linux’，现在是‘git’。    

(在英国俚语中git代表“顽固，自以为是，好辩”)    

此外，作为git的发明者，Linus说到“‘git’可以表示任何含义，完全取决于你的语境。

- 可读的，且没有被任何现有Unix命令使用的3个随即字母的组合。事实上，这是一个“get”的错误发音造成的。   
- “全局信息追踪器”，这次你选对了语境，它确实也会“人如其名”的为你工作。世界一下美好了。   
- “该死的蠢货”，如果它把你的数据搞挂了，你也可以这么理解。
 
### 如何报告git的bug？###
_____________________
可以把bug report发送到git的邮件列表“git@vger.kernel.org”。   
请使用"\[BUG\]"作为邮件标题。你无需订阅即可发布bug report到该邮件列表。参考[GitCommunity](https://git.wiki.kernel.org/index.php)以获取更多关于该邮件列表的信息以及其他和git开发人员交流的方式。
 
### fetch和pull有什么差别？###
_____________________
简单说明如下：   
Fetch：从另外一个代码仓库获取新的对象以及头指针。    
Pull: 首先执行fetch操作，然后将获取到的对象与当前分支merge。    

- 详细信息请参考\[git-fetch\(1\)\]和\[git-pull\(1\)\]的man文档。

### 我可以使用哪些资源来建立一个公共仓库？###
_____________________
一个SSH服务器，或者HTTP服务器亦或是git-daemon。在本地网络甚至可以使用一个贡献的网络文件系统，如NFS或是SMBFS/CIFS(Windows共享)。     
虽然有一些关于使用SMBFS是否安全的争论，但最新的信息表明这应该可行。    

- 查阅Git教程获取详细信息。

### 我可以添加空目录吗？###
_____________________
目前git index的设计只允许列出文件而且尚未有人有能力能修改代码以允许添加空目录。     
当添加文件时，那些文件的父目录会自动被git track。也就是说，目录从来都不需要被显式的添加，git也不会单独去track某个目录。    
当然，你可以使用诸如`git add <dir>`这样的命令，但它实际递归的对该目录所包含的文件执行git add操作。     
如果你的确需要一个空目录在你的工作区中，那么可以在通过那个目录里添加一个.gitignore文件来达到目的。至于.gitignore文件的内容，可以留空，也可以写入你不希望在那个目录中出现的文件的文件名列表。

### 为什么git不记录rename操作？###
_____________________
*Git has to interoperate with a lot of different workflows, for example some changes can come from patches, where rename information may not be available. Relying on explicit rename tracking makes it impossible to merge two trees that have done exactly the same thing, except one did it as a patch (create/delete) and one did it using some other heuristic.*      
另外需要注意的是，追踪rename操作只是追踪目录树中内容变化的一个特例。有时候，有可能更关心某个函数是何时被添加/删除到另外一个文件的。*By only relying on the ability to recreate this information when needed, Git aims to provide a more flexible way to track how your tree is changing.*      
然而，这并不代表Git对追踪rename操作毫无办法。Git中的diff机制支持自动检测rename操作，可以通过`git-diff-*`命令簇中的-M参数来启用该功能。`git-log`和`git-whatchanged`命令使用了rename检测机制。比如，`git log -M`将会输出带有rename信息的提交历史。Git还支持受限形式的跨rename操作的merge。git-blame和git-annotate两个命令也会使用rename检测机制来追踪rename操作。
还有一个非常特出的情况是从git 1.5.3版本开始`git log`命令有一个参数--follow允许你当指定一个特定的文件路径时，追踪rename操作。
      
- [Linus关于这个问题的邮件](http://permalink.gmane.org/gmane.comp.version-control.git/217)     

Git有一个名为`git mv`的rename命令，但那仅仅是为了简化操作。它的实际效果和删除旧文件，添加一个内容相同，文件名不同的新文件无异。

### 为什么git log <filename>操作会慢？###
_____________________
要回答这个问题必须清楚在看似简单的表象背后的逻辑。Git为了找出那些少数修改了指定文件的提交，必须检索全部的提交历史。    
要知道Git不支持基于单个文件的历史。没有文件级的日志决定了你必须检索整个提交历史才能找出那些修改了某个文件的提交。（这样做的原因在于如果Git采用了文件级的日志，那么Git不得不在你要查看全局历史的时候将各个文件的历史排序。）  
一个简单的"git gc"操作有可能会大大提高"git log <filename>"的速度。    
*Note that git log <file1> <file2> (or gitk <file1> <file2>) is not simply the union of git log <file1> and git log <file2>; it can contain merges which are in neither of the separate histories. Doing the history for two files together is not at all equivalent to doing the history for those files individually and stitching it together.*   
可以通过将检索操作限定在一个特定的历史区间来加速`git log`的执行速度，也可以尝试使用--remove-empty参数。    

- Linus Torvalds在邮件列表里关于[如何加速"git log"的讨论](http://permalink.gmane.org/gmane.comp.version-control.git/39358)。

### 为什么export环境变量CDPATH是错误的？ ###
_____________________
CDPATH完全是为了交互式操作设计的。cd命令突然输出一个字符串会导致需要脚本无法正常运行。话虽如此，我们还是做了很多努力来避免问题出现——在所有Git脚本和Makefile中unset CDPATH变量。即使这样，我们还是有可能遗漏了某些细节。  
相比之下，如果你想让自己彻底避免此类问题，你只需将"export"操作从你的.bashrc文件中移除即可。

- [Re: make install bug?](http://article.gmane.org/gmane.comp.version-control.git/13736/match=cdpath)

### 为什么gitweb突然从kernel.org/git上消失了？ ###
_____________________
因为它已经被移植到Git内部。参考[历史提交](http://git.kernel.org/?p=git/git.git;a=commit;h=0a8f4f0020cb35095005852c0797f0b90e9ebb74)

### merge和rebase有什么差别？###
_____________________
假设我们有如下的提交历史：

	- A - B - C - D - remote HEAD	
	    \
	      E - F - G - local HEAD

merge后的历史：

	- A - B - C - D - remote HEAD
    	\                         \
      	 E - F - G - local HEAD - new local HEAD	
rebase后的历史：

	- A - B - C - D - remote HEAD - E' - F' - G' - local HEAD'

可以看到，merge不会改写提交历史，而rebase会。rebase的优势在于提交历史看起来更简洁明了。rebase的缺点在于rebase操作之后原有的E,F,G提交便不存在了，如果此时你发现G中的某些code不能很好的与B中的code协同工作，此时已经无法在回退到G提交了。所谓萝卜白菜，各有所爱，用户需要根据自己的实际情况权衡优劣。

### 为什么git rm不是git add的逆操作？ ###
_____________________
不要想当然的认为'rm'就是'add'的逆操作，那只会让你自己更糊涂！   
当'git add'被用来添加发生在那些已经被Git track的文件上的change到index时，它的逆操作是`git reset HEAD -- <file>`。    
对于添加一个全新文件，它的逆操作是`git rm --cached`。   
看看以下几个命令：

	[1]$ edit a-new-file
	
上边的命令用来创建一个全新文件，此时git还未参与进来。

	[2]$ git add a-new-file
	
这个操作将一个全新文件添加到index中。如果你不想让它出现在index，可以'un-add'它。

	[3]$ git rm --cached a-new-file

这个命令可以将全新文件从index中移除，而不影响工作区中的新文件。如果你想彻底丢弃那个文件，可以继续执行下面的命令：

	[4]$ rm -f a-new-file
	
同样的，这一步中也没git什么事。    
如果你想将第三步和第四步合并为一个操作，你可以使用命令`git rm`（不带--cached参数）。    
显然的，将第一步和第二步合并在一起并无太多实际意义。

- ["Re: git-rm isn't the inverse action of git-add"](http://mid.gmane.org/7vps2y3a4n.fsf@assigned-by-dhcp.cox.net) 来自于Junio C Hamano.
- [git unadd](http://pivotallabs.com/users/alex/blog/articles/1001-git-unadd) 来自于Alex Chaffee

