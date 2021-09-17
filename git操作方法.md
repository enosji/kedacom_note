# 取得项目的GIT仓库

## 目录的初始化

当我们要对某个项目或者文件夹进行git管理的时候使用 `git init` 命令，之后会在该文件夹下面创建一个.git 的目录

## git的三个状态

git管理项目时，将其分为三个工作区域，git本地数据目录（本地仓库）、工作目录、暂存区域。

基本的git工作流程就是：

在工作目录下修改文件，再将这些修改过的文件保存在暂存区域，提交更新后将暂存区域内的文件转存到git目录下

文件在git中存在三种状态：分别为已提交（committed）、已修改（modified）、以暂存（staged） 已提交是指已经将添加的文件安全的保存在本地仓库，已修改表示修改了某个文件，但是没有提交，以暂存表示已经将已修改的文件放入缓冲区准备下次提交

****

# 更新到仓库

## git status

首先确认文件属于什么状态，如果在克隆仓库之后就进行会是以下结果

```c
$ git status
# On branch master
nothing to commit (working directory clean)
```

这说明你的工作区域是赶紧的，这个时候如果我们创建一个新的文件叫README，在`git status`的话会是一下情况

```c
$ git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#	README
nothing added to commit but untracked files present (use "git add" to track)
```

告诉你谁有一个新文件在这个目录下，git没有对他进行追踪，如果需要追踪用git add对其进行追踪

## git add

使用git add来开始跟踪新的文件

```c
git add README
```

这个时候我们再用git status查看，会发现README文件已经被追踪，并处于暂存状态

```c
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   README
#
```

这要在Changes to be committed: 这行下面，就说明是在暂存状态，如果这个时候提交了，此时此刻这个版本就会留存在历史记录里面了，git add 后面可以跟文件名或者目录，如果跟目录就说明要递归跟着这个目录里的所有文件。

## 修改已追踪的文件

当我们修改一个已经在追踪的文件时，例如修改文件benchmarks.rb之后，我们再次运行status命令

```c
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   README
#
# Changed but not updated:
#   (use "git add <file>..." to update what will be committed)
#
#	modified:   benchmarks.rb
#
```

 benchmarks.rb文件出现在Changed but not updated:下面说明文件发生了变化，还没有放入暂存区，需要再次使用add命令（这是个多功能命令，根据目标文件的状态不同，此命令的效果也不同：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等）将这次更新放入暂存区，现在使用add命令

```c
$ git add benchmarks.rb
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   README
#	modified:   benchmarks.rb
#
```

==注意==：如果在你add文件进入缓存区之后有过再次修改文件，必须再次使用add命令，将修改后的重新放入缓存区

## 忽略某些文件

一般来说我们总有些文件无需归纳到git的管理，譬如程序编译过程中产生的中间文件，这时候我们就可以创建一个.gitgnore的文件，列出需要忽略的文件模式，举个简单的例子

```c
$ cat .gitignore
*.[oa]		//告诉git忽略所有的.o和.a结尾的文件，一般这种文件都是编译过程中的中间文件
*~			//告诉git或略所有~结尾的文件
```

.gitgnore文件的格式规格如下

- 所有空行或者以注释符号 ＃ 开头的行都会被 Git 忽略。 
- 可以使用标准的 glob 模式匹配。 * 匹配模式最后跟反斜杠（`/`）说明要忽略的是目录。 *  要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（`!`）取反。 

所谓的 glob 模式是指 shell  所使用的简化了的正则表达式。星号（`*`）匹配零个或多个任意字符；`[abc]`  匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个  c）；问号（`?`）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如  `[0-9]` 表示匹配所有 0 到 9 的数字）。

再看个gitgnore文件例子

```c
# 此为注释 – 将被 Git 忽略
*.a       # 忽略所有 .a 结尾的文件
!lib.a    # 但 lib.a 除外
/TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/    # 忽略 build/ 目录下的所有文件
doc/ *.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```

## git diff

在查看更新的命令中git status的显示比较简单，仅仅是显示修改过的文件，如果要查看具体修改了什么可以用git diff命令进行查看

譬如，我们再次修改README文件后add暂存后，再编辑benchmarks.rb 文件后先别暂存，运行 `status` 命令，会看到：

```c
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   README
#
# Changed but not updated:
#   (use "git add <file>..." to update what will be committed)
#
#	modified:   benchmarks.rb
#
```

要查看尚未暂存的文件更新了那些地方，可以用git diff

```c
$ git diff
diff --git a/benchmarks.rb b/benchmarks.rb
index 3cb747f..da65585 100644
--- a/benchmarks.rb
+++ b/benchmarks.rb
@@ -36,6 +36,10 @@ def main
           @commit.parents[0].parents[0].parents[0]
         end

+        run_code(x, 'commits 1') do
+          git.commits.size
+        end
+
         run_code(x, 'commits 2') do
           log = git.commits('master', 15)
           log.size
```

此命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。

若要看已经暂存起来的文件和上次提交时的快照之间的差异，可以用 `git diff --cached` 命令。（Git 1.6.1  及更高版本还允许使用 `git diff --staged`，效果是相同的，但更好记些。）来看看实际的效果：

```c
$ git diff --cached
diff --git a/README b/README
new file mode 100644
index 0000000..03902a1
--- /dev/null
+++ b/README2
@@ -0,0 +1,5 @@
+grit
+ by Tom Preston-Werner, Chris Wanstrath
+ http://github.com/mojombo/grit
+
+Grit is a Ruby library for extracting information from a Git repository
```

==注意==，单单 `git diff`  不过是显示还没有暂存起来的改动，而不是这次工作和上次提交之间的差异。所以有时候你一下子暂存了所有更新过的文件后，运行 `git diff`  后却什么也没有，就是这个原因。

像之前说的，暂存 benchmarks.rb 后再编辑，运行 `git status` 会看到暂存前后的两个版本：

```c
$ git add benchmarks.rb
$ echo '# test line' >> benchmarks.rb
$ git status
# On branch master
#
# Changes to be committed:
#
#	modified:   benchmarks.rb
#
# Changed but not updated:
#
#	modified:   benchmarks.rb
#
```

现在运行 `git diff` 看暂存前后的变化：

```c
$ git diff 
diff --git a/benchmarks.rb b/benchmarks.rb
index e445e28..86b2f7c 100644
--- a/benchmarks.rb
+++ b/benchmarks.rb
@@ -127,3 +127,4 @@ end
 main()

 ##pp Grit::GitRuby.cache_client.stats 
+# test line
and git diff --cached to see what you’ve staged so far:
$ git diff --cached
diff --git a/benchmarks.rb b/benchmarks.rb
index 3cb747f..e445e28 100644
--- a/benchmarks.rb
+++ b/benchmarks.rb
@@ -36,6 +36,10 @@ def main
          @commit.parents[0].parents[0].parents[0]
        end

+        run_code(x, 'commits 1') do
+          git.commits.size
+        end
+              
        run_code(x, 'commits 2') do
          log = git.commits('master', 15)
          log.size
```

## git commit

现在的暂存区域已经准备妥当可以提交了。在此之前，请一定要确认还有什么修改过的或新建的文件还没有 `git add`  过，否则提交的时候不会记录这些还没暂存起来的变化。所以，每次准备提交前，先用 `git status`  看下，是不是都已暂存起来了，然后再运行提交命令 `git commit`：

```c
$ git commit
```

这种方式会启动文本编辑器以便输入本次提交的说明。（默认会启用 shell 的环境变量 $EDITOR 所指定的软件，一般都是 vim 或  emacs。当然也可以按照第一章介绍的方式，使用 `git config --global core.editor`  命令设定你喜欢的编辑软件。）

第一打开基本上默认的是Nono编译器，这里我们将其改变成vim编译器，打开~/.gitconfig，在文件末尾添加以下内容

```c
$ vi ~/.gitconfig
    
[core]
	editor = vim
```



编辑器会显示类似下面的文本信息（本例选用 Vim 的屏显方式展示）：

```c
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       new file:   README
#       modified:   benchmarks.rb 
~
~
~
".git/COMMIT_EDITMSG" 10L, 283C
```

可以看到，默认的提交消息包含最后一次运行 `git status`  的输出，放在注释行里，另外开头还有一空行，供你输入提交说明。你完全可以去掉这些注释行，不过留着也没关系，多少能帮你回想起这次更新的内容有哪些。（如果觉得这还不够，可以用  `-v` 选项将修改差异的每一行都包含到注释中来。）退出编辑器时，Git 会丢掉注释行，将说明内容和本次更新提交到仓库。

公司的规定格式如下：

```c
第一行：BUG单号/Redmine问题单号，关键字，commit概述
第二行：空行
第三—第N行：详细描述本commit的修改内容、缘由，以及可能产生的影响等
```

![image-20210914112715538](.\git操作方法.assets\image-20210914112715538.png)

也可以使用 -m 参数后跟提交说明的方式，在一行命令中提交更新：

```c
$ git commit -m "Story 182: Fix benchmarks for speed"
[master]: created 463dc4f: "Fix benchmarks for speed"
 2 files changed, 3 insertions(+), 0 deletions(-)
 create mode 100644 README
```

可以看到，提交后它会告诉你，当前是在哪个分支（master）提交的，本次提交的完整 SHA-1  校验和是什么（`463dc4f`），以及在本次提交中，有多少文件修订过，多少行添改和删改过。

记住，提交时记录的是放在暂存区域的快照，任何还未暂存的仍然保持已修改状态，可以在下次提交时纳入版本管理。每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较。



## 跳过暂存区

尽管使用暂存区域的方式可以精心准备要提交的细节，但有时候这么做略显繁琐。Git 提供了一个跳过使用暂存区域的方式，只要在提交的时候，给 `git  commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git  add` 步骤：

```c
$ git status
# On branch master
#
# Changed but not updated:
#
#	modified:   benchmarks.rb
#
$ git commit -a -m 'added new benchmarks'
[master 83e38c7] added new benchmarks
 1 files changed, 5 insertions(+), 0 deletions(-)
```

提交之前不再需要 `git add` 文件 benchmarks.rb 了。



## git rm

要在git中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。可以用 `git rm`  命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

如果只是简单地从工作目录中手工删除文件，运行 `git status` 时就会在 “Changed but not updated”  部分（也就是_ 未暂存_清单）看到：

```c
$ rm grit.gemspec
$ git status
# On branch master
#
# Changed but not updated:
#   (use "git add/rm <file>..." to update what will be committed)
#
#       deleted:    grit.gemspec
#
```

然后再运行 `git rm` 记录此次移除文件的操作：

```c
$ git rm grit.gemspec
rm 'grit.gemspec'
$ git status
# On branch master
#
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       deleted:    grit.gemspec
#
```

最后提交的时候，该文件就不再纳入版本管理了。如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 `-f`（译注：即  force 的首字母），以防误删除文件后丢失修改的内容。

另外一种情况是，我们想把文件从 Git  仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。换句话说，仅是从跟踪清单中删除。比如一些大型日志文件或者一堆 `.a`  编译文件，不小心纳入仓库后，要移除跟踪但不删除文件，以便稍后在 `.gitignore` 文件中补上，用  `--cached` 选项即可：

```c
git rm --cached readme.txt
```

后面可以列出文件或者目录的名字，也可以使用 glob 模式。比方说：

```c
$ git rm log/\*.log
```

注意到星号 `*` 之前的反斜杠 `\`，因为 Git 有它自己的文件模式扩展匹配方式，所以我们不用  shell 来帮忙展开（译注：实际上不加反斜杠也可以运行，只不过按照 shell  扩展的话，仅仅删除指定目录下的文件而不会递归匹配。上面的例子本来就指定了目录，所以效果等同，但下面的例子就会用递归方式匹配，所以必须加反斜杠。）。此命令删除所有  `log/` 目录下扩展名为 `.log` 的文件。类似的比如：

```c
$ git rm \*~
```

会递归删除当前目录及其子目录中所有 `~` 结尾的文件。

## git mv

不像其他的 VCS 系统，Git 并不跟踪文件移动操作。如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。不过 Git  非常聪明，它会推断出究竟发生了什么，至于具体是如何做到的，我们稍后再谈。

既然如此，当你看到 Git 的 `mv` 命令时一定会困惑不已。要在 Git 中对文件改名，可以这么做：

```c
$ git mv file_from file_to
```

它会恰如预期般正常工作。实际上，即便此时查看状态信息，也会明白无误地看到关于重命名操作的说明：

```c
$ git mv README.txt README
$ git status
# On branch master
# Your branch is ahead of 'origin/master' by 1 commit.
#
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       renamed:    README.txt -> README
#
```

其实，运行 `git mv` 就相当于运行了下面三条命令：

```c
$ mv README.txt README
$ git rm README.txt
$ git add README
```

如此分开操作，Git 也会意识到这是一次改名，所以不管何种方式都一样。当然，直接用 `git mv`  轻便得多，不过有时候用其他工具批处理改名的话，要记得在提交前删除老的文件名，再添加新的文件名。

****

# 查看提交历史

## git log

在提交了若干更新之后，又或者克隆了某个项目，想回顾下提交历史，可以使用 `git log` 命令。

打开一个项目，输入git log

```c
$ git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the verison number

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test code

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit
```

默认不用任何参数的话，`git log` 会按提交时间列出所有的更新，最近的更新排在最上面。看到了吗，每次更新都有一个 SHA-1  校验和、作者的名字和电子邮件地址、提交时间，最后缩进一个段落显示提交说明。



## git log -p -2

git log有许多选项可以用，这里就介绍常用的

我们常用 `-p` 选项展开显示每次提交的内容差异，用 `-2` 则仅显示最近的两次更新：

```c
$ git log –p -2
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the verison number

diff --git a/Rakefile b/Rakefile
index a874b73..8f94139 100644
--- a/Rakefile
+++ b/Rakefile
@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
 spec = Gem::Specification.new do |s|
-    s.version   =   "0.1.0"
+    s.version   =   "0.1.1"
     s.author    =   "Scott Chacon"

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test code

diff --git a/lib/simplegit.rb b/lib/simplegit.rb
index a0a60ae..47c6340 100644
--- a/lib/simplegit.rb
+++ b/lib/simplegit.rb
@@ -18,8 +18,3 @@ class SimpleGit
     end

 end
-
-if $0 == __FILE__
-  git = SimpleGit.new
-  puts git.show
-end
\ No newline at end of file
```



## git logn --stat

在做代码审查，或者要快速浏览其他协作者提交的更新都作了哪些改动时，就可以用这个选项。此外，还有许多摘要选项可以用，比如  `--stat`，仅显示简要的增改行数统计：

```c
$ git log --stat 
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the verison number

 Rakefile |    2 +-
 1 files changed, 1 insertions(+), 1 deletions(-)

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test code

 lib/simplegit.rb |    5 -----
 1 files changed, 0 insertions(+), 5 deletions(-)

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit

 README           |    6 ++++++
 Rakefile         |   23 +++++++++++++++++++++++
 lib/simplegit.rb |   25 +++++++++++++++++++++++++
 3 files changed, 54 insertions(+), 0 deletions(-)
```

每个提交都列出了修改过的文件，以及其中添加和移除的行数，并在最后列出所有增减行数小计。



## git log --pretty=

还有个常用的 `--pretty` 选项，可以指定使用完全不同于默认格式的方式展示提交历史。比如用  `oneline` 将每个提交放在一行显示，这在提交数很大时非常有用。另外还有  `short`，`full` 和 `fuller`  可以用，展示的信息或多或少有些不同，请自己动手实践一下看看效果如何。

```c
$ git log --pretty=oneline
ca82a6dff817ec66f44342007202690a93763949 changed the verison number
085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test code
a11bef06a3f659402fe7563abf99ad00de2209e6 first commit
```

有意思的是 `format`，可以定制要显示的记录格式，这样的输出便于后期编程提取分析，像这样：

```
$ git log --pretty=format:"%h - %an, %ar : %s"
ca82a6d - Scott Chacon, 11 months ago : changed the verison number
085bb3b - Scott Chacon, 11 months ago : removed unnecessary test code
a11bef0 - Scott Chacon, 11 months ago : first commit
```

表 2-1 列出了常用的格式占位符写法及其代表的意义。

```c
选项	 说明
%H	提交对象（commit）的完整哈希字串
%h	提交对象的简短哈希字串
%T	树对象（tree）的完整哈希字串
%t	树对象的简短哈希字串
%P	父对象（parent）的完整哈希字串
%p	父对象的简短哈希字串
%an	作者（author）的名字
%ae	作者的电子邮件地址
%ad	作者修订日期（可以用 -date= 选项定制格式）
%ar	作者修订日期，按多久以前的方式显示
%cn	提交者(committer)的名字
%ce	提交者的电子邮件地址
%cd	提交日期
%cr	提交日期，按多久以前的方式显示
%s	提交说明
```

_ 作者（author）_ 和 _ 提交者（committer）_之间究竟有何差别，其实作者指的是实际作出修改的人，提交者指的是最后将此工作成果提交到仓库的人。所以，当你为某个项目发去补丁，然后某个核心成员将你的补丁并入项目时，你就是作者，而那个核心成员就是提交者。我们会在第五章再详细介绍两者之间的细致差别。

用 oneline 或 format 时结合 `--graph` 选项，可以看到开头多出一些 ASCII  字符串表示的简单图形，形象地展示了每个提交所在的分支及其分化衍合情况。在我们之前提到的 Grit 项目仓库中可以看到：

```c
$ git log --pretty=format:"%h %s" --graph
* 2d3acf9 ignore errors from SIGCHLD on trap
*  5e3ee11 Merge branch 'master' of git://github.com/dustin/grit
|\  
| * 420eac9 Added a method for getting the current branch.
* | 30e367c timeout code and tests
* | 5a09431 add timeout protection to grit
* | e1193f8 support for heads with slashes in them
|/  
* d6016bc require time for xmlschema
*  11d191e Merge branch 'defunkt' into local
```

以上只是简单介绍了一些 `git log` 命令支持的选项。表 2-2 还列出了一些其他常用的选项及其释义。

```c
选项 说明
-p 按补丁格式显示每个更新之间的差异。
--stat 显示每次更新的文件修改统计信息。
--shortstat 只显示 --stat 中最后的行数修改添加移除统计。
--name-only 仅在提交信息后显示已修改的文件清单。
--name-status 显示新增、修改、删除的文件清单。
--abbrev-commit 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
--relative-date 使用较短的相对时间显示（比如，“2 weeks ago”）。
--graph 显示 ASCII 图形表示的分支合并历史。
--pretty 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。
```



## 限制输出长度

除了定制输出格式的选项之外，`git log` 还有许多非常实用的限制输出长度的选项，也就是只输出部分提交信息。之前我们已经看到过  `-2` 了，它只显示最近的两条提交，实际上，这是 `-<n>` 选项的写法，其中的  `n` 可以是任何自然数，表示仅显示最近的若干条提交。不过实践中我们是不太用这个选项的，Git  在输出所有提交时会自动调用分页程序（pager），要看更早的更新只需翻到下页即可。

另外还有按照时间作限制的选项，比如 `--since` 和  `--until`。下面的命令列出所有最近两周内的提交：

```c
$ git log --since=2.weeks
```

你可以给出各种时间格式，比如说具体的某一天（“2008-01-15”），或者是多久以前（“2 years 1 day 3 minutes  ago”）。

还可以给出若干搜索条件，列出符合的提交。用 `--author` 选项显示指定作者的提交，用 `--grep`  选项搜索提交说明中的关键字。（请注意，如果要得到同时满足这两个选项搜索条件的提交，就必须用 `--all-match` 选项。）

如果只关心某些文件或者目录的历史提交，可以在 `git log`  选项的最后指定它们的路径。因为是放在最后位置上的选项，所以用两个短划线（`--`）隔开之前的选项和后面限定的路径名。

表 2-3 还列出了其他常用的类似选项。

```c
选项 说明
-(n)	仅显示最近的 n 条提交
--since, --after 仅显示指定时间之后的提交。
--until, --before 仅显示指定时间之前的提交。
--author 仅显示指定作者相关的提交。
--committer 仅显示指定提交者相关的提交。
```

来看一个实际的例子，如果要查看 Git 仓库中，2008 年 10 月期间，Junio Hamano 提交的但未合并的测试脚本（位于项目的 t/  目录下的文件），可以用下面的查询命令：

```c
$ git log --pretty="%h:%s" --author=gitster --since="2008-10-01" \
   --before="2008-11-01" --no-merges -- t/
5610e3b - Fix testcase failure when extended attribute
acd3b9e - Enhance hold_lock_file_for_{update,append}()
f563754 - demonstrate breakage of detached checkout wi
d1a43f2 - reset --hard/read-tree --reset -u: remove un
51a94af - Fix "checkout --track -b newbranch" on detac
b0ad11e - pull: allow "git pull origin $something:$cur
```

Git 项目有 20,000 多条提交，但我们给出搜索选项后，仅列出了其中满足条件的 6 条。

****

# 撤销操作

任何时候，你都有可能需要撤消刚才所做的某些操作。接下来，我们会介绍一些基本的撤消操作相关的命令。请注意，有些操作并不总是可以撤消的，所以请务必谨慎小心，一旦失误，就有可能丢失部分工作成果。

## git commit --amend

修改最后一次提交，有时候我们提交完了才发现漏掉了几个文件没有加，或者提交信息写错了。想要撤消刚才的提交操作，可以使用 `--amend`  选项重新提交：

```c
$ git commit --amend
```

此命令将使用当前的暂存区域快照提交。如果刚才提交完没有作任何改动，直接运行此命令的话，相当于有机会重新编辑提交说明，而所提交的文件快照和之前的一样。

启动文本编辑器后，会看到上次提交时的说明，编辑它确认没问题后保存退出，就会使用新的提交说明覆盖刚才失误的提交。

如果刚才提交时忘了暂存某些修改，可以先补上暂存操作，然后再运行 `--amend` 提交：

```c
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend 
```

上面的三条命令最终得到一个提交，第二个提交命令修正了第一个的提交内容。

## 取消已经暂存的文件

接下来的两个小节将演示如何取消暂存区域中的文件，以及如何取消工作目录中已修改的文件。不用担心，查看文件状态的时候就提示了该如何撤消，所以不需要死记硬背。来看下面的例子，有两个修改过的文件，我们想要分开提交，但不小心用  `git add *` 全加到了暂存区域。该如何撤消暂存其中的一个文件呢？`git status`  命令的输出会告诉你怎么做：

```c
$ git add .
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   README.txt
#       modified:   benchmarks.rb
#
```

就在 “Changes to be committed” 下面，括号中有提示，可以使用 `git reset HEAD  <file>...` 的方式取消暂存。好吧，我们来试试取消暂存 benchmarks.rb 文件：

```c
$ git reset HEAD benchmarks.rb 
benchmarks.rb: locally modified
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   README.txt
#
# Changed but not updated:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   benchmarks.rb
#
```

这条命令看起来有些古怪，先别管，能用就行。现在 benchmarks.rb 文件又回到了之前已修改未暂存的状态。

## 取消对文件的修改

如果觉得刚才对 benchmarks.rb 的修改完全没有必要，该如何取消修改，回到之前的状态（也就是修改之前的版本）呢？`git  status` 同样提示了具体的撤消方法，接着上面的例子，现在未暂存区域看起来像这样：

```c
# Changed but not updated:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   benchmarks.rb
#
```

在第二个括号中，我们看到了抛弃文件修改的命令（至少在 Git 1.6.1  以及更高版本中会这样提示，如果你还在用老版本，我们强烈建议你升级，以获取最佳的用户体验），让我们试试看：

```c
$ git checkout -- benchmarks.rb
$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#       modified:   README.txt
#
```

可以看到，该文件已经恢复到修改前的版本。你可能已经意识到了，这条命令有些危险，所有对文件的修改都没有了，因为我们刚刚把之前版本的文件复制过来重写了此文件。所以在用这条命令前，请务必确定真的不再需要保留刚才的修改。如果只是想回退版本，同时保留刚才的修改以便将来继续工作，可以用下章介绍的  stashing 和分支来处理，应该会更好些。

记住，任何已经提交到 Git 的都可以被恢复。即便在已经删除的分支中的提交，或者用 `--amend`  重新改写的提交，都可以被恢复（关于数据恢复的内容见第九章）。所以，你可能失去的数据，仅限于没有提交过的，对 Git 来说它们就像从未存在过一样。

