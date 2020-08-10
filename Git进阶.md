### 1.关于add和commit指令

之前说过，git把项目本地保存的地方划分为三个分区：工作区、暂存区和版本库。工作区中存放着待编辑文件的副本，也就是在电脑上的项目文件夹里面能看到的原始文件；暂存区中存放着每次提交前的暂存文件，存储路径为.git目录下的index文件，故也被称作index文件；而版本库，则是每次修改后提交的文件存放的地方。add和commit指令分别对应着文件从一个分区移动到另一个分区的过程：文件从工作区添加到暂存区用add指令，文件从工作区提交到版本库用commit指令。我们可以把这个过程类比为一个去超市购物的过程：工作区是超市货架上的商品，暂存区是你自己推的购物车，而版本库是自己家里的储物柜。add就相当于你把一件商品从超市的货架上放入自己的购物车内，commit就是你推着购物车去超市的前台结账，结完账后这些商品自然也就被你拿回家放到了自家的储物柜中。

### 2.关于分支

Git中有一个特别重要的概念：分支（branch）。按照Git官方文档中的说法，分支的引入意味着你可以把自己的工作从开发主线上分离出来，以免影响到开发主线。具体来说，就是我们可以在自己的项目中通过分支的方式建立两条主线：稳定版和测试版。在稳定版的这条线上，我们发布当前项目的稳定版本；而在测试版这条支线上，我们负责日常的开发测试及bug修补，待修补完成后，再把其合并到稳定版。这样，我们的项目就实现的更加安全高效的部署。

### 3.完整步骤

定位到当前目录
`$ cd d:`

从远程仓库下载项目到本地

还有一个与之相似的命令pull，意思是从远程仓库获取最新版本，然后再与本地分支合并，即pull=fetch+merge

`$ git clone https://github.com/MajorHorse/test2.git`
`Cloning into 'test2'...`
`warning: You appear to have cloned an empty repository.`

`$ cd test2`

touch指令创建新文件
`$ touch a.txt`

vi打开并编辑指定文件
`$ vi a.txt`

从工作区转入暂存区
`$ git add a.txt`
`warning: LF will be replaced by CRLF in a.txt.`
`The file will have its original line endings in your working directory`

从工作区提交进仓库
``$ git commit -m 初始版本`
`[master (root-commit) 4e1bbf3] 初始版本`
 `1 file changed, 4 insertions(+)`
 create mode 100644 a.txt`

同步，及将本地代码推送至远程仓库，保持同步
`$ git push`
`Enumerating objects: 3, done.`
`Counting objects: 100% (3/3), done.`
`Writing objects: 100% (3/3), 244 bytes | 244.00 KiB/s, done.`
`Total 3 (delta 0), reused 0 (delta 0), pack-reused 0`
`To https://github.com/MajorHorse/test2.git`

进行第二次修改
`$ `vi a.txt`
`$ git add a.txt`
`warning: LF will be replaced by CRLF in a.txt.`
`The file will have its original line endings in your working directory`
`$ git commit -m 第二次提交`
`[master b27ef6f] 第二次提交 1 file changed, 5 insertions(+), 1 deletion(-)`
`$ git push`
`Enumerating objects: 5, done.`
`Counting objects: 100% (5/5), done.`
`Writing objects: 100% (3/3), 292 bytes | 292.00 KiB/s, done.`
`Total 3 (delta 0), reused 0 (delta 0), pack-reused 0`
`To https://github.com/MajorHorse/test2.git`
   4e1bbf3..b27ef6f  master -> master`

显示所有分支。这里的origin并无明确的具体含义，可以将其理解为一个代号
`$ git branch -a`

* `master     本地master分支`
  `remotes/origin/master     远程master分支`

在本地创建一个名字叫做dev的分支
`$ git branch dev`

显示所有分支
`$ git branch -a`
       `dev     新创建的dev分支`

* `master    主分支master  小点表示当前处于主分支`
  `remotes/origin/master    远程master分支`

删除本地dev分支
`$ git branch -d dev`
`Deleted branch dev (was b27ef6f).`

本地dev分支已删除
`$ git branch -a`

* `master`
  `remotes/origin/master`

-v指令显示提交历史
`$ git branch -v`

* `master b27ef6f 第二次提交`

新建dev分支并显示
`$ git branch dev`

`$ git branch -a`
  `dev`

* `master`
  `remotes/origin/master`

在远程也新建一个名字叫做dev的分支。新建可以简写为git checkout dev origin/dev 
`$ git push origin dev:dev`
`Total 0 (delta 0), reused 0 (delta 0), pack-reused 0`
`remote:`
`remote: Create a pull request for 'dev' on GitHub by visiting:`
`remote:      https://github.com/MajorHorse/test2/pull/new/dev`
`remote:`
`To https://github.com/MajorHorse/test2.git`

 * `[new branch]      dev -> dev`

查看所有分支
`$ git branch -a`
  `dev`

* `master`
  `remotes/origin/dev  远程dev分支已创建`
  `remotes/origin/master`

切换到dev分支下进行工作
`$ git checkout dev`
`Switched to branch 'dev'`

查看日志
$ git log
`commit b27ef6fbfbaa92900ba9206d694ebc2f5cbde2cb (HEAD -> dev, origin/master, origin/dev, master)`
`Author: 子非鱼 <mylove103@foxmail.com>`
`Date:   Mon Jul 27 09:15:03 2020 +0800  第二次提交`

`commit 4e1bbf3054a37c7694026238a660855cd9bb7684`
`Author: 子非鱼 <mylove103@foxmail.com>`
`Date:   Mon Jul 27 09:06:11 2020 +0800  初始版本`

查看所有分支
`$ git branch -a`

* `dev   已经切换到了dev分支下`
  `master`
  `remotes/origin/dev`
  `remotes/origin/master`

对dev分支进行操作

`$ vi a.txt`

`$ git add a.txt`
`warning: LF will be replaced by CRLF in a.txt.`
`The file will have its original line endings in your working directory`

`$ git commit -m 对dev分支的修改`
`[dev 6a8731e] 对dev分支的修改 1 file changed, 6 insertions(+), 1 deletion(-)`

这里之所以报错，是因为远程有俩分支，需要我们明确的指定提交给那个
`$ git push`
`fatal: The current branch dev has no upstream branch.`
`To push the current branch and set the remote as upstream, use`

`git push --set-upstream origin dev`

用它推荐的语句，指定分支并push
`$ git push --set-upstream origin dev`
`Enumerating objects: 5, done.`
`Counting objects: 100% (5/5), done.`
`Delta compression using up to 8 threads`
`Compressing objects: 100% (2/2), done.`
`Writing objects: 100% (3/3), 333 bytes | 333.00 KiB/s, done.`
`Total 3 (delta 0), reused 0 (delta 0), pack-reused 0`
`To https://github.com/MajorHorse/test2.git`
   `b27ef6f..6a8731e  dev -> dev`
`Branch 'dev' set up to track remote branch 'dev' from 'origin'.`

切换到master分支进行操作
`$ git checkout master`
`Switched to branch 'master'`
`Your branch is up to date with 'origin/master'.`

`$ git branch -a`
  `dev`

* `master`
  `remotes/origin/dev`
  `remotes/origin/master`

`$ vi a.txt`
`$ git add a.txt`
`$ giit commit -m 对master分支的修改`
`bash: giit: command not found`
`$ git commit -m 对master分支的修改`
`[master 6a958d1] 对master分支的修改 1 file changed, 1 insertion(+), 1 deletion(-)`
`$ git push --set-upstream origin master`
`Enumerating objects: 5, done.`
`Counting objects: 100% (5/5), done.`
`Delta compression using up to 8 threads`
`Compressing objects: 100% (2/2), done.`
`Writing objects: 100% (3/3), 327 bytes | 327.00 KiB/s, done.`
`Total 3 (delta 0), reused 0 (delta 0), pack-reused 0`
`To https://github.com/MajorHorse/test2.git`
   `b27ef6f..6a958d1  master -> master`
`Branch 'master' set up to track remote branch 'master' from 'origin'.`

合并分支
`$ git merge dev`
`Auto-merging a.txt`
`Merge made by the 'recursive' strategy.`
 `a.txt | 7 ++++++-`
 `1 file changed, 6 insertions(+), 1 deletion(-)`
`$ git branch -a`
  `dev`

* `master`
  `remotes/origin/dev`
  `remotes/origin/maste`r

//同步
`$ git push`
`Enumerating objects: 7, done.`
`Counting objects: 100% (7/7), done.`
`Delta compression using up to 8 threads`
`Compressing objects: 100% (2/2), done.`
`Writing objects: 100% (3/3), 364 bytes | 364.00 KiB/s, done.`
`Total 3 (delta 0), reused 0 (delta 0), pack-reused 0`
`To https://github.com/MajorHorse/test2.git`
   `6a958d1..f57a294  master -> master`

删除dev分支
`$ git branch -d dev`
`Deleted branch dev (was 6a8731e).`

本地dev分支已删除
`$ git branch -a`

* `master`
  `remotes/origin/dev`
  `remotes/origin/master`

删除远程dev分支
`$ git push origin -d dev`
`To https://github.com/MajorHorse/test2.git`

 - `[deleted]         dev`

远程dev分支已删除
`$ git branch -a`

* `master`
  `remotes/origin/master`
