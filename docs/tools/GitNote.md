## 一、Git 介绍

### 1、Git 基础介绍

#### 【1】概念

概念：Git是一个免费的、开源的分布式版本控制系统，可以快速高效的处理从小型到大型的项目

版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统

#### 【2】为什么使用

版本控制工具可以将某个文件回溯到之前的状态，甚至将整个项目都会退到过去某个时间点的状态

就算将项目中的文件改的改删的删。也照样可以恢复到原先的样子

其额外增加的工作量微乎其微，可以对文件的变化细节进行比较，检查出最后是谁修改了哪个地方，从而找出导致怪异问题出现的原因，又是谁在何时报告了某个功能缺陷等等

#### 【3】版本控制系统的分类

集中化的版本控制系统：可能会产生单点故障

<img src="_media/tools/Git/集中化版本控制系统.png"/>

- 集中化的版本控制系统诸如 CVS，SVN 以及 Perforce 等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器,取出最新的文件或者提交更新。
- 多年以来这已成为版本控制系统的标准做法,这种做法带来了许多好处现在,每个人都可以在一定程度上看到项目中的其他人正在做些什么。而管理员也可以轻松掌控每个开发者的权限，并且管理一个集中化的版本控制系统要远比在各个客户端上维护本地数据库来得轻松容易。
- 事分两面，有好有坏，这么做最显而易见的缺点是中央服务器的单点故障。如果服务器宕机一小时，那么在这一小时内，谁都无法提交更新，也就无法协同工作。

分布式的版本控制系统：客户端并不只提取最新版本的文件快照，而是把代码仓库完整的镜像下来

<img src="_media/tools/Git/分布式版本控制系统.png"/>

- 许多这类系统都可以指定和若干不同的远端代码仓库进行交互。这样就可以在同一个项目中分别和不同工作小组的人相互协作
- 分布式的版本控制系统在管理项目时存放的不是项目版本与版本之间的差异，而是历史版本的索引（所需磁盘空间很少，所以每个客户端都可以放下整个项目的历史记录）

### 2、Git 安装

<a href="git-scm.com" target="_blank">官网</a> ：右键 -> Git Bash Here 打开 Git 终端 >> <a href="https://blog.csdn.net/mukes/article/details/115693833" target="_blank">详细安装</a>

### 3、Git 本地结构

<img src="_media/tools/Git/Git本地结构.png"/>

### 4、代码托管中心

#### 【1】代码托管中心的任务

任务是帮助开发人员维护远程库

#### 【2】团队内部协作

<img src="_media/tools/Git/团队内部协作.png"/>

#### 【3】跨团队协作

<img src="_media/tools/Git/跨团队协作.png"/>

#### 【4】托管中心种类

局域网环境下：可以搭建 GitLab 服务器作为代码托管中心

外网环境下：可以使用 GitHub 或 Gitee

### 5、初始化本地仓库

1. 在本地磁盘上创建一个文件夹（ GitResp）
2. 打开 Git 终端 -> GitBash（右键 options，选择 Text ，设置字体和编码）
3. 在 Git 中命令和 Linux 是一样的 ：
   1. 查看 git 安装版本 `git --version`

   2. 清屏 clear

   3. 设置用户名和邮箱：

      > git config --global user.name "zhangsan"
      >
      > git config --global user.email "zhangsan@163.com"
4. 本地仓库的初始化
   1. 在 GitResp 目录下选择 GitBash

   2. `git init` 初始化本地仓库 

   3. `ll -la` ：可以看到 `.git` 隐藏文件

   4. 查看 `.git` 下内容：本地库相关的子目录不要随意删除

      <img src="_media/tools/Git/git文件.png"/>

## 二、常用命令

### 1、add 和 commit 命令

1. 先创建一个文件 -> `demo.txt`
2. 将文件提交到暂存区：`git add demo.txt`
3. 将暂存区文件提交到本地库：`git commit -m "注释：提交文件" demo.txt`
4. 注意：
   1. 如果提交的文件不在 git 本地仓库，git 是无法进行管理的
   2. 即使放在本地仓库的文件，必须通过 add 和 commit 命令，git才会进行管理

### 2、status 命令

git status 看的是工作区和暂存区的状态

当没有文件提交时：

<img src="_media/tools/Git/status1.png"/>

创建一个文件，查看状态：没有被 git 管理

<img src="_media/tools/Git/status2.png"/>

可以通过 `git add Demo2.txt` 将文件提交到暂存区后查看状态：可以被提交

<img src="_media/tools/Git/status3.png"/>

使用 `git commit -m "注释：Demo2" Demo2.txt` 命令提交到本地库

<img src="_media/tools/Git/status4.png"/>

修改 Demo2 文件中内容，并查看：重新添加到暂存区

<img src="_media/tools/Git/status5.png"/>

### 3、log 命令

#### 【1】常用操作

`git log` 可以让我们查看提交的，显示从最近到最远的日志

<img src="_media/tools/Git/log1.png"/>

commit 后面的编号指的是当前历史记录的索引，可以通过索引找到历史版本记录

#### 【2】附加操作

1. 当历史记录过多的时候，查看日志的时候，有分页效果，一页展示不下

   <img src="_media/tools/Git/log2.png"/>

   - 下一页：空格
   - 上一页：b
   - 到尾页，显示 END
   - 退出：q
2. 可以通过 `git log --pretty=oneline` ,表示将日志在一行上展示出来

   <img src="_media/tools/Git/log3.png"/>
3. 通过 `git log -oneline` ，只展示部分索引号的方式

   <img src="_media/tools/Git/log4.png"/>
4. 通过 `git reglog` ，展示指针回到当前这个历史版本需要走多少步

   <img src="_media/tools/Git/log5.png"/>

### 4、reset

#### 【1】reset 指定版本

1. 作用是前进或者后退历史版本
2. 若要将版本从当前版本退回到上一个版本

   <img src="_media/tools/Git/reset1.png"/>

   - `git reset --hard 67c8975`

     <img src="_media/tools/Git/reset2.png"/>

#### 【2】hard 参数（常用）

本地库的指针移动的同时，重置暂存区，重置工作区

#### 【3】mixed 参数

本地库的指针移动的同时，重置暂存区，工作区不动

#### 【4】soft 参数

本地库的指针移动的时候，暂存区和工作区都不动

### 5、删除文件、找回本地库删除的文件

#### 【1】删除文件

1. 前置：新建一个 Test2.txt 文件，将它 add 到暂存区中，再通过 commit 提交到本地库
2. 删除工作区中 Test2.txt 文件：`rm Test2.txt` 
3. 将删除操作同步到暂存区：`git add Test2.txt `
4. 将删除操作同步到本地库：`git commit -m "删除 Test2.txt 文件" Test2.txt `
5. 查看日志 `git reflog`

   <img src="_media/tools/Git/删除文件.png"/>

#### 【2】找回本地库删除的文件

实际上就是将历史版本切换到刚才添加文件的那个版本即可

### 6、找回暂存区删除的文件

1. 前置：删除工作区中 Test2.txt 文件：`rm Test2.txt `
2. 将删除操作同步到暂存区：`git add Test2.txt `
3. 恢复暂存区数据：与找回本地库删除的文件相同操作 `/ git reset --hard HEAD`

### 7、diff 比对文件差异

1. 前置：新建一个 Test3.txt 文件并输入一些内容，将它 add 到暂存区中，再通过 commit 提交到本地库
2. 更改工作区中 Test3.txt 中的内容，此时工作区和暂存区中的数据不一致
3. 通过 `git diss Test3.txt` 来比对该文件工作区和暂存区中的数据

   <img src="_media/tools/Git/diff1.png"/>
4. 当多个文件比对时，`git diff` 比较工作区中和暂存区中所有的文件进行比较
5. 比较暂存区和工作区中差别： `git diff [历史版本/HEAD] [文件名]`

## 三、分支

### 1、什么是分支

在版本控制过程中，使用多条线同时推进多个任务，这里的多条线就是多个分支

分支的好处在于可以并行咖啡，互不影响，互补耽误，提高开发效率，如果有一个分支功能开发失败，那么直接删除即可，不会对其他分支产生影响

<img src="_media/tools/Git/分支.png"/>

### 2、查看、创建、切换分支

1. 前置：前置：新建一个 Test4.txt 文件并输入一些内容，将它 add 到暂存区中，再通过 commit 提交到本地库
2. 通过 `git branch -v `查看当前所有分支

   <img src="_media/tools/Git/查看分支.png"/>
3. 通过 `git branch branch01` 创建分支

   <img src="_media/tools/Git/创建分支.png"/>
4. 再次查看分支

   <img src="_media/tools/Git/再次查看分支.png"/>
5. 通过 `git checkout branch01` 切换分支

   <img src="_media/tools/Git/切换分支.png"/>

### 3、冲突问题

在同一个文件的同一个位置修改，会出现冲突问题

2. 在 branch01 分支中增加 Test4.txt 文件内容，将它 add 到暂存区中，再通过 commit 提交到本地库
3. 将分支切换到 `master：git checkout master`
4. 在 master 分支中增加 Test4.txt 文件内容，将它 add 到暂存区中，再通过 commit 提交到本地库
4. 再次切换到  branch01 分支中

   <img src="_media/tools/Git/合并冲突解决.png"/>
5. 将 branch01 分支合并到 master 分支
   1. 进入主分支：`git checkout master`
   2. 通过 `git merge branch01` 将 branch01 中的内容和主分支内容进行合并

      <img src="_media/tools/Git/合并冲突.png"/>
   3. 查看出现冲突的文件：cat Test4.txt

      <img src="_media/tools/Git/查看冲突文件.png"/>
   4. 公司内部商议解决，或者自己决定，人为决定。留下想要的即可，之后重新提交到暂存区

      <img src="_media/tools/Git/冲突解决重新add.png"/>
   5. 进行 commit 操作

      <img src="_media/tools/Git/冲突解决commit.png"/>

## 四、GitHub

### 1、注册 GitHub 账号

GitHub 官网：github.com

<img src="_media/tools/Git/GitHub注册.png"/>

### 2、初始化本地库

1. 前置：新建一个文件夹 `mkdir GitResp2 -> cd GitResp2 `
2. `git init`
3. 新建文件 Demo.txt ，将它 add 到暂存区中，再通过 commit 提交到本地库

### 3、创建 GitHub 远程库

<img src="_media/tools/Git/创建GitHub远程仓库1.png"/>

<img src="_media/tools/Git/创建GitHub远程仓库2.png"/>

<img src="_media/tools/Git/创建GitHub远程仓库3.png"/>

### 4、在本地创建远程库地址的别名

1. 获取远程库的地址：

   <img src="_media/tools/Git/获取远程库地址1.png"/>

   ![image-20220526143241660](F:\Java\images.assets\Git.assets\获取远程库地址2.png"/>
2. 远程库地址比较长，每次复制比较麻烦，在 Git 本地中可以通过别名将地址保存
3. 查看别名

   <img src="_media/tools/Git/查看别名.png"/>
4. 起别名

   <img src="_media/tools/Git/起别名.png"/>

### 5、推送操作

通过 git push origin master 推送到远程仓库（git push [别名] [分支名]）

<img src="_media/tools/Git/推送到远程.png"/>

### 6、克隆操作

通过 `git clone [远程库地址]` 进行克隆操作

<img src="_media/tools/Git/远程库复制.png"/>

<img src="_media/tools/Git/克隆.png"/>

克隆操作可以完成：
1. 初始化本地库
2. 将远程库内容完整的克隆到本地
3. 创建远程库的别名

### 7、邀请加入团队 push 操作

1. 前置：新建 Demo2.txt 文件，将它 add 到暂存区中，再通过 commit 提交到本地库
2. 将文件 push 到远程库：`git push orgin master` ，此时模拟的是团队其他人提交代码，所以提交是应该重新输入账号密码，但因这个账号没有加入到远程仓库的团队中，所以提交失败

   <img src="_media/tools/Git/提交失败.png"/>
3. 需要登录有权限的账号进行邀请

   <img src="_media/tools/Git/邀请1.png"/>

   <img src="_media/tools/Git/邀请2.png"/>

   <img src="_media/tools/Git/复制邀请链接.png"/>
4. 更换到被邀请的账号，在地址栏粘贴邀请链接

   <img src="_media/tools/Git/接受邀请.png"/>

### 8、远程库修改的拉取

#### 【1】fetch + merge

1. 拉取操作相当于 `fetch + merge`
2. 项目经理先确认远程库内容是否更新了
3. 项目经理进行拉取
4. 通过 `git fetch origin master` 抓取远程库内容（只是下载到本地，没有更新到工作区）

   <img src="_media/tools/Git/fetch.png"/>
5. 抓取后可以去远程库看看内容是否正确

   <img src="_media/tools/Git/查看远程库内容.png"/>
6. 通过 `git merge origin/master` 进行合并操作，合并前要将分支切换回来

   <img src="_media/tools/Git/合并.png"/>

#### 【2】pull

通过 git pull origin master 直接拉取

<img src="_media/tools/Git/pull.png"/>

fetch + merge 操作为了保险慎重

当代码简单时可以使用 pull

### 9、协同开发时冲突解决

<img src="_media/tools/Git/协同开发冲突.png"/>

在冲突的情况下，应该先拉取下来，修改冲突后，再推送到远程服务器

`git pull origin master`

<img src="_media/tools/Git/冲突文件.png"/>

解决后重新提交

### 10、免密登录

1. 进入用户的主目录中 cd~
2. 执行命令，生成一个 .ssh 目录：`ssh-keygen -t rsa -C [github邮箱]`

   <img src="_media/tools/Git/免密登录.png"/>
3. 在  .ssh 目录下有两个文件

   <img src="_media/tools/Git/ssh.png"/>
4. 打开 id_rad.pub 文件复制内容
5. 在 GitHub 中 settings，找到 `SSH and GPG keys`

   <img src="_media/tools/Git/sshkeys.png"/>

## 五、IDEA集成Git

### 1、IDEA 集成 Git

#### 【1】初始化本地库

IDEA 集成 Git 

<img src="_media/tools/Git/idea集成git.png"/>

本地初始化操作

<img src="_media/tools/Git/初始化.png"/>

#### 【2】添加暂存区

<img src="_media/tools/Git/ideaAdd.png"/>

#### 【3】提交本地库操作

<img src="_media/tools/Git/ideaCommit.png"/>

### 2、使用 IDEA 拉取和推送资源

拉取：使用 `git pull origin master --allow-unrekated-histories` （允许历史内容合并）（`git pull [地址] [分支名] --allow-unrekated-histories`）

<img src="_media/tools/Git/push.png"/>

推送：`git pull -u origin master -f`

<img src="_media/tools/Git/ideapush.png"/>

在 IDEA 中推送

<img src="_media/tools/Git/在idea中推送.png"/>

<img src="_media/tools/Git/在idea中推送2.png"/>

### 3、使用 IDEA 克隆远程仓库到本地

<img src="_media/tools/Git/IDEAclone.png"/>

<img src="_media/tools/Git/ideaCLone2.png"/>

克隆到本地后，这个目录既变成了一个本地仓库，又变成了工作空间

### 4、使用 IDEA 解决冲突

#### 【1】解决冲突

<img src="_media/tools/Git/推送被拒绝.png"/>

在 push 后，存在冲突，推送被拒绝，需要先合并

<img src="_media/tools/Git/merge.png"/>

<img src="_media/tools/Git/idea合并.png"/>

#### 【2】避免冲突

团队开发的时候避免在一个文件中改代码

在修改一个文件前，在 push 之前，先 pull 操作