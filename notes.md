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

## lesson-2 了解Branch

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

## lesson 3 了解Merge

Merge的目的就是将不同的分支功能合并

合并规则:

+ 将两个分支，分别相对祖先节点标注出相对的修改部分，当其中一个分支相对修改位置在另外一个分支没有修改，或者两个分支的修改相同，则允许合并，不然则冲突

执行```git merge 被合并的分支名(feature)```将当前分支(cur)与分支(feature)进行合并

**合并后:**

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

reset有三种模式:

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

这里要重新解释一下:

+ 暂存区对相同文件只会保留最后一次add

+ 工作区其实和暂存区很像，但是git工作区会保留未提交的修改，比如git reset --mixed HEAD^后，暂存区回到未add的状态，工作区回到保存了未提交的修改的状态

## lesson 5 reflog

这个命令的作用是查看HEAD曾经指向的位置，返回的形式如下:

+ ```commit-hash HEAD@{时间}: 操作说明``` 

+ 之后按 q 退出

但注意，过了保存期限的Commit对象可能不会显示

当想要恢复时，就执行 git reset --hard 恢复的Commit对象的hash 即可，这个命令实际过程如下:

+ 当前分支指针指向目标Commit对象，'回'遍历目标节点，直到将目标节点的父节点按顺序最终和reset前的Commit对象连接起来

+ 这意味着，见下例:

reset前: A \to B(cur) \to C, D \to E(目标节点)

reset后: A \to B \to D \to E, C(悬空)