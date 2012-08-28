### 3 Unexpected behavior ###
#### 3.1 Why won't I see changes in the remote repo after "git push"?
#### 3.2 Why is git --version not reporting the "full" version number?
#### 3.3 Why is "git reset --hard" not removing some files?
#### 3.4 Why is my push rejected with a non-fast forward error?
#### 3.5 Why won't "git push" work after I rebase a branch?
#### 3.6 Why is "git commit -a" not the default?
#### 3.7 My HTTP repository has updates, which git clone misses. What happened?
#### 3.8 Why isn't Git preserving modification time on files?
#### 3.9 Why does git use a pager for commands like diff/log and --help?
#### 3.10 Why does diff/log not show color, even though I enabled it?
#### 3.11 Why does git diff sometimes list a file that has no changes?
#### 3.12 What does the gitk error message "Can't parse git log output:" mean?
#### 3.13 Why does gitk on Cygwin display "git 1316 tty_list::allocate: No tty allocated"?
#### 3.14 Why does git clone, git pull, etc. fail when run on a partition mounted with sshfs (FUSE)?
#### 3.15 Why does "git bisect" make me test versions outside the "good-bad" range?
#### 3.16 Why am I "not on any branch"?
#### 3.17 "git log -S" does not show all commits

### 为什么`git push`后在远程仓库看不到我的change？###
____________________
push操作会将本地的提交历史推送到远程仓库并更新refs，而不会更改工作区的文件。具体来说，如果你push本地change到一个远程仓库已经checkout的分支，这并不会改变远程仓库中工作区的文件。     
这样的行为是经过考虑的。因为在你执行push操作时，远程仓库中的工作区可能已经有了change。这时，你作为一个向远程push change的人员，是无法解决你push的change与远程仓库工作区change的冲突的（*注:因为冲突会发生在工作区，你无法直接操作一个远程仓库的工作区*）。然而，你可以通过一个post-update hook来更新一个已经checkout的分支的工作区，但问题在于这样的hook在出现问题时只会告知执行push操作的用户(*注:远程仓库的owner对此一无所知*)。最新的post-update hook的草稿在[http://utsl.gen.nz/git/post-update](http://utsl.gen.nz/git/post-update)。它基本可以处理我们说过的所有情况，除了以下两种情况————远程仓库的工作区中已然存在了由merge引起的conflict，或者是你要push的change跟远程仓库当前checkout的分支根本就没有冲突。    
千言万语一句话，__永远不要__push change到一个non-bare的仓库，除非你真的知道你在干什么。     
如果你非要这么干，你可以在远程仓库执行一个`git reset --hard`操作来摒弃远程仓库工作区中的change。这会让你丢到在远程仓库中尚未提交的所有修改，然后checkout你push过去的最新的提交。关于bare仓库的信息可以参考[这篇文章](http://sitaramc.github.com/concepts/bare.html#how_do_I_fix_such_a_non_bare_push_)。    

### 为什么`git --version`不显示完整的版本号？### 
____________________
这就好比是先有鸡还是先有蛋的问题。当你在build Git源码时，编译程序需要一个可用的git命令来获取当前code的完整版本号(*注:显然build之前是没有的，否则你也不需要build了*)。如果在你已经git后，再从源码build一次，这是新的binary就可以知道它的完整版本号了。    
脚本GIT_VERSION_GEN可以输出当前git的版本，而git --version显示的是在build源码时使用的版本号。

### 为什么`git reset --hard`没有删除某些文件？###
____________________
`git reset`不会删除它不track的问题，包括那些被告知ignore的文件。如果reset操作切换到一个包含较旧版本.gitignore文件的提交，那么有可能某些之前被ignore的文件会在`git status`中显示为未被track。

- Linus Torvalds关于此问题的[邮件](http://marc.theaimsgroup.com/?l=git&m=114917892328066)；Sean Estabrooks关于此问题的[邮件](http://marc.theaimsgroup.com/?l=git&m=114917892613019)。
- 如果你使用的是cygwin或OSX，那你可能会遇到大小写敏感问题————参考[讨论](http://marc.info/?l=git&w=2&r=1&s=xt_CONNMARK.h&q=b)。

### 为什么我的push失败了，提示我"non-fast forward"？###
____________________
如果在尝试push一个分支，你可能得到了类似下面的错误信息：
>! [rejected]        master -> master (non-fast forward)   
>error: failed to push some refs to 'git@github.com:pieter/gitx.git'

该信息表明你的本地分支不是远程分支严格意义上的超集。换句话说，远程分支包含你本地分支中没有的提交。如果在这种情况下push可能会导致丢失提交。通常情况下，你只需要在本地执行以下命令：
	
	git pull
	
你也可以先通过以下命令查看本地分支中没有的远程提交：

	git fetch origin
	git log master..origin/master
	
如果你想通过图形界面查看这些提交可以使用`gitk --left-right master...origin/master`。指向左边的箭头表示你想要push的change，指向右边的箭头表示远程分支拥有的提交。    
如果尝试push一个rebase过的分支，请直接参考下一小结。   
如果你知道你在做什么，你可以尝试：

	git push origin +branchname
	
这个命令会强制更新远程分支。如果你没有权限强制push，你可以尝试下面两个命令：

	git push origin :branchname
	git push origin +branchname
	
这两个命令会先将远程分支删除，再重新建立新分支。    
需要注意的是在你强制更新远程分支后，其他用户在pull时可能会遇到问题。还有一种可能是其他用户可能会将你丢弃的提交再次引入到该分支上。基于这些原因，改变一个分支的提交历史通常是不可取的。尽管如此，它并非一无是处。

### 为什么在我rebase一个分支后`git push`会失败？###
____________________
在你rebase一个本地分之后，你尝试push你的change到远程仓库。这时，git push命令失败并告诉你以下错误信息。
> error: remote 'refs/heads/master' is not a strict subset of local ref 'refs/heads/master'. maybe you are not up-to-date and need to pull first?

这不是bug，而是必要的安全检查：`git push`不会更新远程分支，除非远程分支上最新的提交出现在当前本地分支的提交历史中。这项检查可以避免你将其他用户在你最后一次fetch后推送到远程仓库的提交覆盖。如果没有这项检查，其他用户的提交很有可能丢失。这项检查避免了你不小心使用一个本地分支覆盖掉远程一个毫不相干的分支。    
当你rebase时，你以rebase时使用的新base作为嫁接点修改了当前分支的提交历史。这就导致了你rebase后，远程分支最新提交将不再是本地分支最新提交的父提交。此时，如果你执行`git push`意味着你要用新的提交历史改变替换掉远程仓库中可能已经被其他用户fetch的提交历史。      
如果你确实需要这么做，你可以使用`git push -f`。然而，最近版本的Git中已经默认禁用了push -f操作，因为那通常都是给你制造麻烦，而不是解决。如果你一定需要强制推送到远程仓库，那么你可以通过在远程仓库中运行以下命令来允许强制push：

	git config receive.denyNonFastForwards false
	
在强制推送change到远程仓库后，__切记__将此配置项重置为true。

### 为什么`git commit -a`不是默认的？###


