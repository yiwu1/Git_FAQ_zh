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