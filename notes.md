# Git 学习笔记

## lesson-1 一些基础知识

Git有三个区:1.工作区;2.暂存区;3.仓库

**命令:**

+ git init :在当前目录初始化一个Git仓库(即创建隐藏的.git文件夹，包含底层的对象数据库)

+ git status :查看当前工作区，暂存区状态

+ git add :把文件加入暂存区

**存入暂存区的原理阐述:**

当Git把文件加入暂存区，Git会读取文件全部内容，加上一个头部信息，然后通过SHA-1算法生成一个40位的哈希值

接着，Git把该文件压缩生成一个对象，存放在.git/objects目录下，文件路径就是这个哈希值(如哈希值:1234，文件路径则为12\34)，这种专门存储文件内容的对象就是Blob对象

接着，Git会更新暂存区文件(.git/index)，在其中记录下一次commit的候选快照

注意！Blob对象只存储文件内容不存储文件名！也就是说当把文件复制一份，改名并git add，Git底层只会生成一个Blob对象

**命令:**

+ ls -la :ls表示list，-l表示列出长信息，-a表示列出隐藏文件，-la即表示-l+-a

+ git ls-files --stage :查看暂存区

**commit提交的原理阐述:**

暂存区是一张平铺的表，当我们commit的时候，Git会把这个表打包成一个Tree对象，存入.git/objects

Tree对象保存了当前暂存区的目录结构，文件名和文件名所指向的Blob对象的哈希值

接着，Git会生成一个Commit对象，它很小，只包含一些关键信息

有:

+ 指向刚刚Tree对象的哈希值

+ 作者和提交者信息

+ 提交说明

+ 父提交，即指向前一次的提交，是Git能把提交历史记录连起来

**命令:**

+ git config --local user.name "..." :设置当前作者名

+ git config --local user.email "..." :设置当前作者邮箱

+ git commit -m "提交信息"

+ git log :查看仓库提交历史，展现一串Commit对象的哈希值

+ git cat-file -p :cat-file表示打印Git的底层对象，-p表示"漂亮的打印"，即将对象的解压缩再打印，当传入一个哈希(40位或者简短的前7位)，它就能查看该对象

+ git log --oneline :查看commit日志

**knowledge:**

+ 1.master是Git底层的文本指针，它始终指向最后一次commit的Commit对象

+ 2.HEAD是git底层的文本指针，它指向当前工作的哪一次commit状态

## lesson-2 了解branch

Bracn相当于是一个指向Commit对象的可变文本指针(实际存储分支名字+Commit引用)，相关的有以下命令:

+ git branch 分支名字 :创建分支文本指针，当前与master指向同一个Commit对象

+ git checkout 分支名字 :让HEAD指针指向分支指针

+ git checkout -b 分支名字 :创建分支指针并立即让HEAD指向该分支指针

+ git branch :查看本地所有分支，带*的是HEAD所指向的

注意，新提交的Commit对象只会导致HEAD所指向的分支指针移动！最终形成像树一样的Commit对象连接结构

重新回顾:命令git log --oneline的作用是从当前头指针HEAD一路打印回最初的commit记录，是一个子路！

当我们修改且未保存的时候，修改是在编辑器内存空间上进行的，未改变磁盘空间，切换HEAD指针指向不会进行覆盖(改内存空间独立)，只用当保存后才会认为你在当前指向的Commit对象下有新的修改

当我们进行了新的修改，之后我们切换分支的时候，Git会进行如下审查:

+ Git审查两个分支，判断出当前未提交修改的部分，也判断出目标分支相对当前分支的修改部分

+ 如果目标分支相对当前分支的修改部分与当前未提交修改有重叠，即会覆盖当前未提交修改的部分，则不允许切换

+ 可以用命令git checkout -f 分支名字 强行切换，这样会直接用目标分支的文件覆盖当前分支工作区的文件

## lesson 3 了解merge

merge的目的就是将不同的分支功能合并

执行```git merge 被合并的分支名(feature)```将当前分支(cur)与分支(feature)进行合并

**合并规则:**

+ Case1:如果cur所指向的是feature的父节点，则直接挪动cur指针到feature指针处

+ Case2:反之如果feature指针指向cur指针所指向的的父节点，则无事发生，因为cur处可是视作feature的发展了

+ Case3:反之，产生新的Commit对象，其有两个父节点，分别为feature指针所指向的和cur指针所指向的，然后挪动cur指针指向当前新的Commit对象

**命令:**

+ git log --graph --all :展现出分支图

+ git merge 被合并分支名 -m "合并说明" 

+ git show 哈希值 :展现对象属性和内容

补充说明:git log --oneline后，如果遇到多个父节点，他会将父节点依次遍历，遍历所有分支到父节点们的公共父节点后在继续遍历

**这里解释一下怎么在linux子系统里面运行C++文件:**

+ first step: g++ 文件名 -0 生成的可执行文件名

+ second step: ./可执行文件名

## lesson 4 了解reset

reset就是回滚指针

**reset有三种模式:**

+ 1. git reset --soft

+ 2. git reset --mixed

+ 3. git reset --hard

三种移动方式造成的影响不同，主要是影响移动后工作区，暂存区是否同步，注意reset都是对当前分支指针的操作

方式1: 会回到指定的commit状态，但是工作区和暂存区仍保留reset前分支指针的状态，可以用来修改commit信息

```
git reset --soft HEAD^
git commit -m "重新修改的信息"
```

方式2: 同方式1，但是暂存区会回到指定commit状态下的工作区，也就是说可能需要重新git add

方式3: 同方式1，但是工作区和暂存区都会回到指定commit状态

当reset后，目标节点和中间连接的节点变为不可达状态(没有分支指针能回滚到它)，Git有垃圾回收机制，当这些节点长时间不可达时，会被删除

**这里要重新解释一下:**

+ 暂存区对相同文件只会保留最后一次add

+ 工作区其实和暂存区很像，但是Git工作区会保留未提交的修改，比如git reset --mixed HEAD^后，暂存区回到未add的状态，工作区回到保存了未提交的修改的状态

## lesson 5 了解reflog

这个命令的作用是查看HEAD曾经指向的位置，**返回的形式如下:**

+ ```commit-hash HEAD@{时间}: 操作说明``` 

+ 之后按 q 退出

但注意，过了保存期限的Commit对象可能不会显示

当想要恢复时，就执行 git reset --hard 恢复的Commit对象的hash即可，这个命令实际过程如下:

+ 当前分支指针指向目标Commit对象，'回'遍历目标节点，直到将目标节点的父节点按顺序最终和reset前的Commit对象连接起来

+ 这意味着，见下例:

reset前: $A \to B(cur) \to C, \quad D \to E(目标节点)$

reset后: $A \to B \to D \to E, \quad C(不可达)$

注意虽然HEAD指向的是分支指针，但是它会通过解析分支指针的指向来记录移动

**reflog也可以指定分支，HEAD和每个分支都有自己的reflog记录**

```git reflog 分支名```可以用来查看分支的reflog记录

##  lesson 6 了解remote

remote 本质上是存储了本地仓库与云端github上仓库的联系

**命令:**

+ git remote -v :查看当前本地仓库与github上的仓库的联系

+ git remote add origin 仓库地址 :向与github上的仓库建立联系，其中 origin 是这个联系的名字

在以下讲解中，用 origin 充当联系的名字

**origin/cur:**

这是本地保存的远程状态记录，如 $A \to B(origin/cur) \to C$ 意味着本地Git认为云端仓库的最新状态是origin/cur所处的位置

## lesson 7 了解push

**命令:**

+ git push -u origin 本地分支名cur0:远程分支名cur :上传的同时，建立跟踪联系，即之后Git会默认将origin/cur移动到cur0处

+ git push -u origin cur0 :可以省略本地分支名，这样会默认选择同名分支，特别的，如果远程仓库不存在cur0分支或者与cur0联系的分支，则远程仓库会创建cur0分支，origin内多出origin/cur0分支

+ git push origin cur :将origin/cur移动到与其联系的cur0处(无联系则默认位置，即本地同名分支，但注意不会创建本地同名分支与origin分支的联系！)

一般来说， git push origin cur相当于reset origin/cur的位置，Git会拒绝覆盖远程仓库记录的push，即**Git要求origin/cur是cur0的祖先**，如:

```
本地: A \to B(cur0)
远程: A \to B \to C(origin/cur)
git push origin cur -- 被拒绝

本地: A \to B \to C(cur0)
远程: A \to B \to D(origin/cur)
git push origin cur -- 被拒绝
```

注意，push不仅仅是将Commit对象上传到远程仓库，它还会将其相关的Tree对象，Blob对象也上传

push和fetch一样，会把远程仓库不存在的该分支的可达对象上传

## lesson 8 学习fetch和pull

### fetch部分

fetch的工作: 默认情况下，对远程仓库所有分支cur，下载远程仓库相对于origin/cur的新的内容，并更新origin/cur的位置

注意，默认下，fetch不是针对某一个origin的分支指针而言，它会更新所有的origin分支指针

**命令:**

+ git fetch origin :对origin的分支进行如上更新，注意不会改变本地的分支指针

+ git fetch origin main : 获取远程仓库的main分支，移动origin/main

第一次fetch，会加载远程仓库，建立与远程仓库一样的branch图

但是fetch的要求宽松:

当我们每一次fetch，本地都会记录远程仓库分支结构，同时将本地origin下的分支移动到远程仓库的对应的新位置

当我们指定分支(main)时，Git会检查远程仓库main分支的可达对象，是否有需要补充的对象，然后挪动origin/main，如:

```
origin: A \to B \to C(origin/main)
remote: A \to D(main)

after fetch: origin: A \to D(origin/main)
```

### pull部分

pull相当于fetch + merge

**命令:**

+ git pull origin main :先执行git fetch origin main，然后执行git merge origin/main(合并origin/main和本地HEAD指向的分支指针)

+ git pull origin :先执行git fetch origin，只会按照origin分支与本地分支之间的联系进行merge，无联系则会报错

注意，要遵循merge的规则