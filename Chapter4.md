### 4 How do I ... ###
#### 4.1 How do I specify what ssh key git should use?
#### 4.2 How do I untrack a file?
#### 4.3 How do I access other branches in a repository?
#### 4.4 How do I share a git public repository and use it in a CVS way?
#### 4.5 How do I share a git repository using POSIX ACLs?
#### 4.6 How can I add a diff of the commit into the commit message window?
#### 4.7 How would I use "git push" to sync out of a host that I cannot pull from?
#### 4.8 How do I check out the tree at a particular tag?
#### 4.9 How to share objects between existing repositories?
#### 4.10 How to stop sharing objects between repositories?
#### 4.11 How do I tell git to ignore files?
#### 4.12 How do I tell git to ignore tracked files?
#### 4.13 How do I save some local modifications?
#### 4.14 How to manually resolve conflicts when Git failed to detect rename?
#### 4.15 How to revert file to version from current commit?
#### 4.16 How to view an old revision of a file or directory?
#### 4.17 How can I tweak the date of a commit in the repo?
#### 4.18 How do I make a diff between two arbitrary files in different revisions?
#### 4.19 How to fix a broken repository?
#### 4.20 How to remove all broken refs from a repository?
#### 4.21 How to set up a git server?
#### 4.22 How to create the first project?
#### 4.23 How do I publish my repo via SFTP?
#### 4.24 How do I do a quick clone without history revisions?
#### 4.25 How do I use git for large projects, where the repository is large, say approaching 1 TB, but a checkout is only a few hundred MB? Will every developer need 1 TB of local disk space?
#### 4.26 How do I obtain a list of files which have changed in a given commit?
#### 4.27 How do I remove all the old objects after using filter-branch?
#### 4.28 How do I mirror a SVN repository to git?
#### 4.28.1 Troubleshooting
#### 4.29 How do I edit the root commit?
#### 4.30 How do I clone a subdirectory?
#### 4.31 How do I make existing non-bare repository bare?
#### 4.32 How do I find large files?
#### 4.33 How do I recover uncommitted changes?
#### 4.34 How do I merge a patch from a gmane.org URL?
#### 4.35 How do I clone a repository with all remotely tracked branches?
#### 4.36 How do I remove my uncommitted changes from branch A and add them to a new branch B?
#### 4.37 How do I use external diff tools?

### 如何为Git设置一个专用的ssh key？###
__________________
确切的说，这并不是一个Git的问题。然而，你可以编辑~/.ssh/config来告诉ssh使用哪个key来连接Git服务器。关于此配置的详细信息请参考[ssh配置手册](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh_config)。但是简单来说你可以使用下面的配置项为GitServer设置一个单独的key:

>Host GitServer    
>  Hostname=git.example.org    
>  IdentityFile=~/.ssh/my_cool_key_rsa

然后在下次你通过git链接GitServer时，指定的key就会被使用。

### 如何untrack一个文件？###
__________________
如果你想在提交历史中保留某个文件，但是在下一次提交中移除它，你可以使用命令:

	git rm --cached <filename>
	
### 如何访问仓库中的其他分支？###
__________________
当clone一个仓库时，Git会为远程仓库中的每一个分支在本地建立一个remote tracking分支。但是只会建立一个本地分支，通常是"master"。你可以使用`git branch -r`命令查看所有本地的remote tracking分支：

>Vienna:git pieter$ git branch -r  
>  origin/HEAD  
>  origin/html  
>  origin/maint  
>  origin/man  
>  origin/master  

这些remote tracking分支代表了在你clone时，远程仓库中分支的状态。你基本可以把他们当成本地分支对待，比如你可以`git log origin/maint`，但是有一点需要注意，就是`git checkout`命令。如果你checkout到一个remote tracking分支，那么你会进入"分离头指针"状态。因此如果你想在那些分支商进行开发，你需要建立一个本地的分支：

>Vienna:git pieter$ git checkout -b maint origin/maint  
>Branch maint set up to track remote branch refs/remotes/origin/maint.   
>Switched to a new branch "maint"   
>Vienna:git pieter$   

以上命令会创建一个名为"maint"的本地分支，该分支会指向和origin/maint同样的提交，然后你就可以在本地的maint分支上进行开发了。

### 如何创建一个公共的Git仓库，然后以CVS的方式使用它？###
__________________
你可以使用命令`git --bare init --shared=group`（或是`git --bare init --shared=all`）来初始化一个公共仓库。所有属于你group中的用户均可以push change到此仓库。*It's O.K. that refs aren't group writable, it's enough that the directory is。*

- 参考Git [cvs迁移文档](http://www.kernel.org/pub/software/scm/git/docs/gitcvs-migration.html)中的"Emulating the CVS Development Model"一节。

### 如何使用POSIX ACL来共享一个Git仓库？###
__________________
你可以使用setfacl命令来给单个用户授权。假设你的账户是"alice"，你希望给"charlie"和"bob"授权。可以使用以下命令：

	setfacl -Rm u:bob:rwx '/home/alice/path/to/repo'
	setfacl -Rm u:charlie:rwx '/home/alice/path/to/repo'
	
接下来需要设置默认ACL以保证新创建的文件拥有同样的授权：

	setfacl -Rm d:u:bob:rwx '/home/alice/path/to/repo'
	setfacl -Rm d:u:charlie:rwx '/home/alice/path/to/repo'
	setfacl -Rm d:u:alice:rwx '/home/alice/path/to/repo'
	
如果需要，还要设置相应的目录权限：

	setfacl -m u:bob:x '/home/alice/path/to'
	setfacl -m u:charlie:x '/home/alice/path/to'
	setfacl -m u:bob:x '/home/alice/path'
	setfacl -m u:charlie:x '/home/alice/path'
	setfacl -m u:bob:x '/home/alice'
	setfacl -m u:charlie:x '/home/alice'	
	
如果你的umask设置限制了group访问(比如umask077)那么1.7.0及之前版本的Git会破坏你的ACL并创建不可读的pack文件。想要避免此问题可以将仓库设置为共享：

	git config core.sharedRepository group
	
### 如何在提交信息编辑对话框中加入此次提交的diff内容？###
只需给`git commit`命令加上-v参数：

	git commit -v
	
### How would I use "git push" to sync out of a host that I cannot pull from?#
__________________
如果你工作在两个各自拥有自己工作区的仓库(两仓库分布在不同主机)上，通常保持两个仓库同步的方法就是在每个仓库中pull对方。然而，由于某些限制，你可能只能从一台主机像另一台主机建立TCP链接，反之则不可以。假设你在主机A上创建一个项目，然后在主机B上clone了它。你在B上进行开发并想将change带回到A。你不能在A上执行`git fetch`因为主机B的防火墙不允许进入的TCP链接。这种情况下该如何做呢？   
如果你能意识到push和fetch操作的本质，你就能利用它来解决这个问题。如果B没有防火墙限制，你可能会选择在A上fetch B。这时的fetch操作实质上是将B中master分支同步到A中的refs/remotes/B/master上。   
然而，在B上将master分支push到A中的master分支显然不是我们需要的操作。因为push是一个fetch的对等操作，它会将本地的提交同步到远程仓库，但不会改变远程仓库的工作区。如果你这么做，当你回到主机A上时会发现master分支已经指向新的提交，但是工作区并没有相应的更新。    
因此，解决这种防火墙问题的简单方法是从B上push master分支到A上的refs/remotes/origin/master，而不是refs/heads/master：

	machineB$ git push machineA:repo.git master:refs/remotes/B/master
	
当你回到主机A上时，刚才的命令就好像是你执行了下面的命令：

	machineA$ git fetch machineB:repo.git master:refs/remotes/B/master
	
这与你在A上fetch B没有任何差别。你还可以编辑.git/config文件设置默认的push行为。这样每次你只需:

	git push
	
就可以完成以上操作。

- 参考关于此问题的[讨论](http://thread.gmane.org/gmane.comp.version-control.git/42506/focus=42685)

### 如何checkout特定tag下的目录树？###
__________________
使用`git tag -l`查看本地tag，然后使用`git checkout TAGNAME`即可。如果你想在那个tag上进行开发，那么使用`git checkout -b newbarnch TAGNAME`来创建一个新分支以开始工作。如果你想回到之前的分支，可以使用`git checkout ORIGINALBRANCH`，通常是master分支。

### 如何在不同仓库之间贡献object？###
__________________
使用以下2个命令：

	echo "/source/git/project/.git/objects/" > .git/objects/info/alternates
	git repack -a -d -l
	
-l参数意味着repack命令仅将本地object打包进pack文件(确切的说，它也会将alternate tree中的对象打包。这样你就有了一个完整的pack文件包含了所有对象)。

### 如何停止在不同仓库间共享object？###
__________________
使用以下2个命令：

	git repack -a
	rm .git/objects/info/alternates
	
如果在这两个命令之间仓库被修改，那么当alternates文件被删除时可能会导致数据不一致。如果你不确定，可以使用`git fsck`来检查是否有数据不一致出现。如果出现问题，你可以将alternates文件恢复并重新执行以上2命令。

### 如何让Git忽略某些文件？###
__________________
你可以在.git/info/exclude或.gitignore文件中添加SHELL样式的glob语句来告诉Git忽略匹配的文件。
.git/info/exclude只工作在本地，也不会通过push，pull来与他人共享。
.gitignore使用的更为普遍，而且可以作为普通文件被添加到Git仓库并与其他人共享。

- 参考[git-ls-files](http://www.kernel.org/pub/software/scm/git/docs/git-ls-files.html)的man文档中"Exclude Patterns"一节。

### 如何让Git忽略已经track的文件？###
__________________
这种情况通常发生一个项目中的示例配置文件上，你可能想在他本地的代码库中修改这个文件但又不想提交这些更改更不想将这些更改与他人共享。但是如果你不提交这些change，那么这些change会在每次你运行`git diff`时被显示出来，而且还很容易被一个不经意的`git commit -a`或是`git add -u`添加到代码仓库中。    
要避免这个问题有3种解决方案：

- 方案1 将配置文件重命名为config.template。然后告诉用户使用它作为示例配置文件。
- 方案2 允许在其他文件中修改配置。比如，在Makefile中可以加参数-include config.mak来允许用户在config.mak文件中写入定制的配置。
- 方案3 使用Git来忽略这些配置文件。没有一种能实现是可以通过仓库共享给所有用户的，你只能在每个本地仓库中实现这个效果。你可以使用`git update-index --assume-unchanged <file>`或是sparse checkout([参考git-read-tree中相关章节(http://www.kernel.org/pub/software/scm/git/docs/git-read-tree.html)])来达到目的。需要注意的是设计这些命令的初衷并不是为了解决这个问题，因此，小心点。

### 如何临时保存一些本地修改？###
__________________
有时你需要将一些当前分支上未提交的change放在一边，去到另外一个分支上进行开发。但是由于某些原因，你暂时还不想将这些change提交。    
在最近版本的Git中，你可以使用`git stash`临时保存修改，稍后在使用`git stash apply`重新应用这些修改。    
此外，   
你还可以创建一个临时分支来保存这些修改：

	$ git checkout -b tempBranch
	$ git commit -a -m "to test"
	
另外一个方法就是把这些修改保存成一个patch文件：

	$ git diff --binary HEAD > tempPatch.diff
	$ git reset --hard
	
### 当Git无法检测到rename时，如何手工解决冲突？###
__________________
当你修改了一系列文件，但merge却因为不能正确检测rename而无法自动解决冲突时，该如何做呢？假设你有一个util/endian.h文件，在开发过程中，你将它移动到src/util/endian.h，而你的同事继续使用util/endian.h。终于，要merge两个分支的时刻到了，有时git的recursive merge策略可以检测到你移动了文件，并将你同事在util/endian.h上的修改merge到src/util/endian.h。但事情并不总是这么顺利，Git可能会认为你删除了util/endian.h并创建了一个不相关的src/util/endian.h文件。你会得到一个util/endian.h上的冲突信息"your side removed, other side modified"。    
首先使用`git ls-files --unmerged`查看哪些文件有冲突。然后你就可以看到每个stage状态下的blob对象的ID.stage1来自common ancestor，stage3来自你同事的分支。当这种类型的冲突发生时，你不会看到stage2.因为Git认为你将util/endian.h文件删除了：
>$ git ls-files --unmerged --abbrev    
>…   
>100755 33cd1f76... 1 util/endian.h     
>100755 7f531bb7... 3 util/endian.h    

接着在这些文件之间手工merge：   
>$ git cat-file blob 33cd1f76 >endian.h-1     
>$ git cat-file blob 7f531bb7 >endian.h-3      
>$ merge src/util/endian.h endian.h-1 endian.h-3      

当然你可以使用任何其他的三方合并工具，比如Ediff3，vimdiff/gvimdiff，Meld，xxdiff或KDiff3。在新版本的Git中，你可以使用"<stage>:<filename>"格式的字符串替换通过blob ID解压出来的临时文件。      
一旦你完成合并得到了你需要版本的src/util/endian.h，你就可以通知Git这个文件被重命名了。在我们的例子中可以使用以下两个命令：

	git update-index --remove util/endian.h
	git update-index src/util/endian.h
	
### 如何将文件恢复到当前提交中的版本？###
__________________
如果你将一个文件搞乱了，或者不小心删掉了，你想将这个文件恢复到当前提交中的版本，你可以使用:

	git checkout HEAD -- <file>
	
如果你想恢复到index中该文件的版本，你可以：

	git checkout -- <file>
	
### 如何查看一个较早版本的文件或目录？###
__________________
使用`git show`命令：

	git show <commit>:path/file
	
<commit>可以使提交ID，分支名，tag等。如果你不指定文件名，它会输出指定提交下顶层目录树中的内容。一些命令示例：   

	git show v1.4.3:git.c   
	git show f5f75c652b9c2347522159a87297820103e593e4:git.c     
	git show HEAD~2:git.c     
	git show master~4:      
	git show master~4:doc/     
	git show master~4:doc/ChangeLog      

### 如何修改仓库中某个提交的提交日期？###
__________________
如果你想修改HEAD的提交日期，最简单的方法是使用下面的命令：      

	$ git commit --amend --date='<see documentation for formats>' -C HEAD

你可以使用date命令方便的得到新的时间：

	$ current_date=$(git log -1 --format=format:%ai)
	$ new_date=$(date -d "$current_date - 1 hour - 22 minutes" --rfc-2822)
	$ git commit --amend --date="$new_date" -C HEAD	
如果你还想设置一个新时区，那么你还要设置TZ这个环境变量。如果要将时区改为太平洋时间，你需要将new_date=…一行改为：

	$ new_date=$(TZ=:US/Pacific date -d "$current_date - 1 hour - 22 minutes" --rfc-2822)
	
如果你只是想修改一下时区，那么使用Git内部保存时间时使用的Unix时间戳格式会更简单。比如，如果你要HEAD中提交时间的时区为UTC，你可以：

	$ current_date=$(git log -1 --format=format:%at)
	$ git commit --amend --date="$new_date +0000" -C HEAD
	
你可以像下面这样将Unix时间戳传递给date命令：

	$ current_date=$(git log -1 --format=format:%at)
	$ TZ=:US/Pacific date -d @1301500895
	Wed Mar 30 09:01:35 PDT 2011
	
如果你不记得当前的时区，没关系，问问date命令：

	$ date +%z
	+0000
	
如果你想知道当前时区和某个时区(比如美国太平洋时间)的时差，试试下面的命令：

	$ TZ=:US/Pacific date +%z
	<either -0700 or -0800 depending on the current date>
	$ TZ=:US/Pacific date -d 2011-01-01 +%z
	-0800
	$ TZ=:US/Pacific date -d 2011-08-01 +%z
	-0700
	
如果你想修改某个HEAD的父提交，只需使用`git bisect -i`来选择在rebase时你想暂停下来修改的提交。如果你需要一个更正式的方式，你可以使用x指令来为`git rebase -i`设置在指定提交上执行的命令。以下是使用`git fileter-branch`命令来实现此操作的脚本：

	#!/bin/sh
	# Rewrite all branches to modify the date of one specific commit in a repo.
	# Sample date format: Fri Jan 2 21:38:53 2009 -0800
	# ISO8601 and RFC822 dates will also work.
	# Note: filter-branch is picky about the commit argument. As of 1.7.0.4
	# a hex ID will work, the symbolic revision HEAD will fail silently,
	# and the usability of more exotic rev specs was not tested by the author.
	# Copyright (c) Eric S. Raymond, 2010-08-01. BSD terms apply (if anybody really thinks that this script is long and non-obvious enough to fall under copyright law).
	#
	commit="$1"
	date="$2"
	git filter-branch --env-filter \
    	"if test \$GIT_COMMIT = '$commit'
     		then
     			export GIT_AUTHOR_DATE
	           export GIT_COMMITTER_DATE
         		GIT_AUTHOR_DATE='$date'
		       GIT_COMMITTER_DATE='$date'
     	 fi" &&
	rm -fr "$(git rev-parse --git-dir)/refs/original/"
	
