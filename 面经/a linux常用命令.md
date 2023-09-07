# cmake

### makefile

​	make 是一个命令工具，是一个解释 makefile 中指令的命令工具。

​	make 工具在构造项目的时候需要加载一个叫做 makefile 的文件，makefile 关系到了整个工程的编译规则。一个工程中的源文件不计数，其按类型、功能、模块分别放在若干个目录中，makefile 定义了一系列的规则来指定**哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译**，甚至于进行更复杂的功能操作，因为 makefile 就像一个 Shell 脚本一样，其中也可以执行操作系统的命令。

​	makefile 带来的好处就是 —— `自动化编译`，一旦写好，只需要一个 make 命令，整个工程完全自动编译，极大的提高了软件开发的效率。

### cmake

> ​	cmake可以更加简单的生成makefile文件给make用，根据一个叫CMakeLists.txt文件（学名：组态档）去生成makefile。
>
> ​	补充：CMake是一个跨平台工具，因为在windows上没有Makefile，所以就需要一个同时**适用于linux和windows的自动编译脚本**，所以有了CMakelists.txt，这个文件是linux和windows通用的，利用cmake，可以根据CMakelist.txt在不同的操作系统上产生不同的自动编译脚本文件。

​	这里使用了cmake，因为文件比较多。在项目目录中写好CMakeLists.txt后，之后有源文件的改动，只需要更改CMakeLists.txt中的几行即可

* 1、写好CMakeLists.txt文件

* 2、创建build文件夹，在build目录中cmake

  ```shell
  mkdir build
  cd build
  cmake -G"MinGW Makefiles" ..
  # .. 表示 CMakeLists.txt 在上一级目录。
  # Windows 下，CMake 默认使用微软的 MSVC 作为编译器，我想使用 MinGW 编译器，可以通过 -G 参数来进行指定，只有第一次构建项目时需要指定。
  ```

* 此时在 build 目录下会生成 Makefile 文件，然后调用编译器来实际编译和链接项目：`cmake`



makefile编译，保存在了build文件夹下，编译为libevent.so动态库和libevent.a静态库，供给其他程序作为网络库使用

通常系统在编译时会在./usr/lib和/usr/local/lib这两个文件夹下找.a(静态库) .so(动态库)文件，在/usr/include和/usr/local/include文件夹下找.h文件。当然，这个.h文件也可以放在项目下，反正保证`#include "xx.h"能找到就行`

make中通过install把.so库放到了/usr/local/lib文件夹下，把threadpool.h放到了/usr/local/include/文件夹下

[cmake教程](https://zhuanlan.zhihu.com/p/500002865)





当然也可以使用如下命令，将当前目录下所有文件都参与编译

```cpp
aux_source_directory(. SRC_LIST)
```

但增加新文件后，makefile还是原来的，需要右键cmakelists.txt文件夹，选择clean reconfigure all projects，然后重新cmake出makefile，再rebuild就行了。但还是有点麻烦，不如直接`set(SRC_LIST xxx.cc)`，每次写上新的.cc文件就多写一个set





src中的CMakeLists

```makefile
#aux_source_directory(. SRC_LIST)
set(SRC_LIST 
    myrpcapplication.cc 
    myrpcconfig.cc 
    rpcheader.pb.cc 
    rpcprovider.cc 
    myrpcchannel.cc
    myrpccontroller.cc
    logger.cc
    zookeeperutil.cc)
add_library(myrpc ${SRC_LIST}) # 生成myrpc库
target_link_libraries(myrpc muduo_net muduo_base pthread zookeeper_mt)
```



步骤：

* 设置cmake的最低版本和项目名称
* 通过set设置好项目根目录、源码目录、头文件目录等变量
* file(GLOB xx 目录）命令在指定的目录内匹配到所需要的文件，然后将他们set到一个变量里面
* 后面add_library(SHARED)还是add_library(STATIC)就都用这个变量了
* target_link_libraries链接库
* 最后install安装共享库和头文件到系统目录，才能真正让其他人开发使用

```shell
# 删减版
# cmake教程 https://zhuanlan.zhihu.com/p/500002865
# 设置cmake的最低版本和项目名称
project(WebServer LANGUAGES CXX)
cmake_minimum_required(VERSION 3.5.1)

#项目根目录
set(ROOT_DIR ${CMAKE_CURRENT_LIST_DIR})
#源码目录
set(SRC_DIR ${ROOT_DIR}/src)
#头文件路径
set(INC_DIR ${ROOT_DIR}/include)
#库文件路径
set(LIB_DIR /usr/lib/x86_64-linux-gnu)
#可执行文件安装路径
set(EXEC_INSTALL_DIR ${ROOT_DIR}/bin)
#库安装路径
set(LIB_INSTALL_DIR ${ROOT_DIR}/lib)


# CPP源文件
# file GLOB命令主要用于匹配规则在指定的目录内匹配到所需要的文件，命令行格式：
file(GLOB UTILITY_SRC_FILE        ${SRC_DIR}/utility/*.cpp)
file(GLOB THREAD_SRC_FILE         ${SRC_DIR}/thread/*.cpp)
file(GLOB TIMER_SRC_FILE          ${SRC_DIR}/timer/*.cpp)
set(MAIN_SRC_FILE                 ${ROOT_DIR}/main.cpp)

set(LIBEVENT_SOURCES ${UTILITY_SRC_FILE}
                     ${THREAD_SRC_FILE}
                     ${TIMER_SRC_FILE}
                     ${SERVER_SRC_FILE})

#头文件路径添加
include_directories(${INC_DIR})
#库文件路径添加
link_directories(${LIB_DIR})
#用到的库
set(LINK_LIBRARY pthread)

#生成动态链接库，库的类型为SHARED即动态链接库
add_library(event_shared SHARED ${LIBEVENT_SOURCES})
#更改动态链接库输出名字
set_target_properties(event_shared PROPERTIES OUTPUT_NAME event)
#设置动态链接库版本号
set_target_properties(event_shared PROPERTIES VERSION 1.2 SOVERSION 1)
#链接库
target_link_libraries(event_shared ${LINK_LIBRARY})


# install安装共享库和头文件到系统目录，才能真正让其他人开发使用
# 将.a和.so文件放到lib文件夹，将.h文件放到include文件夹
install(TARGETS event_shared event_static DESTINATION ${LIB_INSTALL_DIR})
install(TARGETS web_server DESTINATION ${EXEC_INSTALL_DIR})
```



# git

> reference：[廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600) [阿秀](https://mp.weixin.qq.com/s?__biz=Mzg2MDU0ODM3MA==&mid=2247509037&idx=1&sn=417030295af768482c87a3210f357a21&chksm=ce265850f951d1463f3411a9b096ade58264a93660cb0523c68c0847bfcfff07ab780532015b&scene=178&cur_album_id=1857548892041478149#rd)

​	Git是目前世界上最先进的**分布式版本控制系统**。

​	git的使用需要先创建一个版本库，可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

> 注意，git只能跟踪文本文件的改动，比如TXT文件，网页，所有的程序代码等等；对于图片、视频等只能知道比如大小从100KB到120KB这种，不会知道改了啥。也不能跟踪Word



> 平时是怎么用git的？

​	目前的话，我自己的使用场景有两个。一个是维护我的几个项目到github上，这个比较简单，主要是项目有改动的时候git add，git commit，git push。另一个使用场景就是我把我markdown的笔记保存在了github上，然后在实验室、公司、个人笔记本三端进行一个同步。这里因为是我个人用，所以就维持了一个main分支，保持github仓库是最新版本。前期需要先将每个电脑的公钥添加到github上，然后clone一下。三端使用的时候都需要git pull origin main拉取最新，然后进行了一些修改后，再add commit push。

​	要是多人使用的场景下，就需要commit pull push。其中pull是为了本地 commit 和远程commit 的记录进行对比，git 是按照文件的行数操作进行对比的,如果同时操作了某文件的同一行那么就会产生冲突，git 也会把这个冲突给标记出来，这个时候就需要先把和你冲突的那个人拉过来问问保留谁的代码，然后在 git add && git commit && git pull，再次 pull 一次是为了防止再你们协商的时候另一个人给又提交了一版东西，如果真发生了那流程重复一遍，通常没有冲突的时候就直接给你合并了，不会把你的代码给覆盖掉。

## git命令

**git init** 

​	将当前目录变成Git可以管理的仓库

**git remote** 

* git remote add xxx git@github.com:yzew/learngit.git 关联远程库，xxx是远程库的名字

* git remote -v 查看远程库的信息

**git diff**

​	查看自己对代码做出的改变，也就是查看暂存区与disk区文件的差异。

**git add** xxx

​	将文件添加到暂存区

**git commit** -m "说明" 

​	提交

**git status**

​	查看工作区状态

**git log**

​	显示从最近到最远的提交日志

**git checkout**

* -b xxx 创建并切换到xxx分支

**git switch**

* git switch -c xxx 创建并切换到xxx分支（git checkout xxx同样功能）
* git switch xxx 切换到xxx分支（git checkout -- xxx）

**git reset**

* git reset HEAD xxx 把暂存区的修改撤销掉，重新放回工作区
* `git reset --hard HEAD^` 回退版本（cmd中`git reset --hard "HEAD^"`，用powershell就没这个问题）
  (在Git中，用`HEAD`表示当前版本，也就是最新的提交，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本就是`HEAD~100`)

**git rm** xxx

​	从版本库中删除文件

**git push** -u 远程库名 分支名

​	把本地库的xxx分支的所有内容推送到远程库上的同名分支。第一次推送要带着-u，后面就不需要了

**git clone** 地址

​	将github上的库克隆到本地

**git branch**

* git branch查看当前是哪个分支
* git branch -d xxx 删除xxx分支（-D是强制删除）

**git merge** xxx

​	将xxx分支合并到当前分支上

**git pull** 远程库名 分支名

​	从远程主机下载远程分支并与本地同名分支合并。

​	git pull --rebase xxx xxx:不再出现自动生成的 merge commit，提交树会保持一条直线

> `git add .`：会提交当前工作区中当前目录(包括子目录)下所有的文件改动。将所有新文件和对已有文件的改动以及被删除的文件提交至暂存区
>
> 如果想在使用 `git add .` 时不提交被删除的文件，可以使用 `git add --ignore-removal` 加上匹配符 `.` 即 `git add --ignore-removal .`
>
> `git add .` 提交的文件改动受当前所在目录限制，它只会提交当前工作区中当前目录(包括子目录)下的文件改动，而 `git add -A` 不受当前所在目录的限制，提交的是当前整个工作区中所有的文件改动。同样可以 `git add --ignore-removal -A`
>
> `git add *` 表示添加当前目录(包括子目录)下的所有文件改动，但不包括文件名以 `.` 符号开头的文件的改动。这是 Shell 命令，git 只是接收文件列表。而 `git add .` 的功能与 `git add *` 基本相同，只是 `git add .` 会将文件名以 `.` 符号开头的文件的改动也提交至暂存区。
>
> 通常用`git add .`比较多，其包括了.开头的文件。
>
> 对于.开头的文件，比如.gitignore文件可以设置一些过滤规则，比如不将一些本地配置信息等推送到服务器中

## git操作

**将文件添加到版本库**

​	文件必须放到仓库下(如我的gitstore文件夹)，然后在git仓库目录内执行git命令：

* 将文件添加到仓库：git add readme.txt (可反复多次使用，添加多个文件)
* 把文件提交到仓库：git commit -m "wrote a readme file" (`-m`后面输入的是本次提交的说明)(一次可以提交多个文件)
  <img src="E:\MarkDown\picture\image-20230527194716049.png" alt="image-20230527194716049" style="zoom:60%;" />
  `1 file changed`：1个文件被改动（我们新添加的readme.txt文件）；`2 insertions`：插入了两行内容（readme.txt有两行内容）。



**修改文件并添加到版本库**

* 对文件的内容进行更改
* 使用`git status`命令查看工作区状态：
  <img src="E:\MarkDown\picture\image-20230527195126773.png" alt="image-20230527195126773" style="zoom:67%;" />
  上面的命令输出告诉我们，`readme.txt`被修改过了，但还没有准备提交的修改。
* 可以使用`git diff`查看修改了什么内容
  <img src="E:\MarkDown\picture\image-20230527195504954.png" alt="image-20230527195504954" style="zoom:67%;" />
* 对文件改动后，`git add`、`git commit`添加到仓库中
  <img src="E:\MarkDown\picture\image-20230527195742037.png" alt="image-20230527195742037" style="zoom:70%;" />



**版本回退**

* `git log`命令显示从最近到最远的提交日志
  <img src="E:\MarkDown\picture\image-20230527200005884.png" alt="image-20230527200005884" style="zoom:67%;" />

* `git reset`命令回退版本：`git reset --hard HEAD^`（注意，在cmd中换行符默认是`^`，而不是`\`，因此要执行`git reset --hard "HEAD^"`，用powershell就没这个问题）
  (在Git中，用`HEAD`表示当前版本，也就是最新的提交，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本就是`HEAD~100`)
  <img src="E:\MarkDown\picture\image-20230527200255721.png" alt="image-20230527200255721" style="zoom:80%;" />
* 回退之后，此时再用`git log`再看看现在版本库的状态，就会发现之前最新的修改记录已经没有了
  <img src="E:\MarkDown\picture\image-20230527200642503.png" alt="image-20230527200642503" style="zoom:70%;" />
* 如果又想改回去，只要上面的命令行窗口还没有被关掉，就可以顺着往上找，找到那个`append GPL`的`commit id`是`e71390...`，于是就可以指定回到未来的某个版本：
  <img src="E:\MarkDown\picture\image-20230527200939329.png" alt="image-20230527200939329" style="zoom:80%;" />
* 如果找不到新版本的`commit id`，可以通过命令`git reflog`查看之前记录的每一次命令：
  <img src="E:\MarkDown\picture\image-20230527201106106.png" alt="image-20230527201106106" style="zoom: 67%;" />



**工作区和暂存区**

<img src="E:\MarkDown\picture\image-20230619192834879.png" alt="image-20230619192834879" style="zoom:67%;" />

**工作区**（Working Directory）

​	就是在电脑里能看到的目录，比如我的`gitstore`文件夹就是一个工作区

**版本库**（Repository）

​	工作区有一个隐藏目录`.git`，这个不算工作区，而是Git的版本库。

​	Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的**暂存区**，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。

前面讲了我们把文件往Git版本库里添加的时候，是分两步执行的：

第一步是用`git add`把文件添加进去，实际上就是把文件修改添加到**暂存区**；

第二步是用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。

因为我们创建Git版本库时，Git自动为我们创建了唯一一个`master`分支，所以，现在，`git commit`就是往`master`分支上提交更改。

一旦提交后，如果你又没有对工作区做任何修改，那么工作区就是“干净”的，即用`git status`后显示：On branch master nothing to commit, working tree clean



**撤销修改**

如果仅是在工作区（就是还没有add，仅改了下文件）进行了修改，想恢复到上一个版本的状态，可以使用 `git checkout -- readme.txt` 将`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。



如果是在add后想恢复，此时修改只是添加到了暂存区，还没有提交，可以通过`git reset HEAD <file>`把暂存区的修改撤销掉，重新放回工作区。`git reset`命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用`HEAD`时，表示最新的版本。

此时暂存区就是干净的了，而工作区有修改。再`git checkout -- xx.txt`，工作区的修改也恢复了。



如果是commit后想恢复，就是前面提到的版本回退了



**删除文件**

将文件在工作区rm后，git status可以发现某些文件被删除了

* 确实要从版本库中删除该文件，那就用命令`git rm xxx.txt`删掉，并且`git commit`
* 删错了，此时可以很轻松地把误删的文件恢复到最新版本：`git checkout -- xxx.txt`



## github仓库

先有本地库，后有远程库的时候，如何关联远程库：

​	登陆GitHub，在右上角找到 `New repository` 按钮，创建一个新的仓库，比如这里创建了一个learngit

```shell
# 在本地目录下创建个git仓库
git init

# 关联
# 关联一个远程库时必须给远程库指定一个名字，origin是默认习惯命名
git remote add origin git@github.com:yzew/MarkDown.git


# 这里默认本地有个分支了。如果没有的话：
git add .
git commit -m "threadpool first c"

# 注意，github 为我们的新仓库创建main作为默认的branch
# 而git创建的默认分支叫Master
# 因此需要将本地仓库的master分支重命名为main分支
git branch -m master main
# 注意，这里通过git config --global init.defaultBranch main命令已经设置了以后创建的项目默认分支为main，以后就不用改分支名了

# 把本地库的所有内容推送到远程库上。第一次推送要带着-u参数
git push -u origin main

# 以后提交就用这个：
git push origin main

```



先创建远程库，然后从远程库克隆：

github创建一个远程库

```shell
# 克隆到本地
git clone git@github.com:yzew/learngit.git
```

## 分支管理

> 详细流程看https://www.liaoxuefeng.com/wiki/896043488029600/900003767775424

​	你创建了一个属于你自己的分支，别人看不到，还继续在原来的分支上正常工作，而你在自己的分支上干活，想提交就提交，直到开发完毕后，再一次性合并到原来的分支上，这样，既安全，又不影响别人工作。

​	在Git里，主分支即`master`分支。`HEAD`严格来说不是指向提交，而是指向`master`，`master`才是指向提交的，所以，`HEAD`指向的就是当前分支。

​	一开始的时候，`master`分支是一条线，Git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支，以及当前分支的提交点：

<img src="E:\MarkDown\picture\image-20230623151200910.png" alt="image-20230623151200910" style="zoom:67%;" />

​	之后每次提交，`master`分支都会向前移动一步，这样，随着你不断提交，`master`分支的线也越来越长。

​	当我们创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上：

<img src="E:\MarkDown\picture\image-20230623151234845.png" alt="image-20230623151234845" style="zoom: 67%;" />

​	从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变：

<img src="E:\MarkDown\picture\image-20230623151339560.png" alt="image-20230623151339560" style="zoom:67%;" />

​	假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。Git怎么合并呢？最简单的方法，就是直接把`master`指向`dev`的当前提交，就完成了合并：

<img src="E:\MarkDown\picture\image-20230623151404404.png" alt="image-20230623151404404" style="zoom:67%;" />

​	合并完分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支：

<img src="E:\MarkDown\picture\image-20230623151436088.png" alt="image-20230623151436088" style="zoom:67%;" />

​	若是`master`分支和`feature1`分支各自都分别有新的提交commit，变成了这样：

<img src="E:\MarkDown\picture\image-20230623155756206.png" alt="image-20230623155756206" style="zoom:67%;" />

​	此时git merge xxx时，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突。必须手动解决冲突后再提交。

```shell
git merge feature1
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
```

​	此时直接修改xxx文件中的内容，Git用`<<<<<<<`，`=======`，`>>>>>>>`标记出不同分支的内容。修改后再提交，`master`分支和`feature1`分支变成了下图所示：

<img src="E:\MarkDown\picture\image-20230623160552386.png" alt="image-20230623160552386" style="zoom:67%;" />

## 多人协作

> https://www.liaoxuefeng.com/wiki/896043488029600/900375748016320

多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。



看到这了https://www.liaoxuefeng.com/wiki/896043488029600/900003767775424









# vim

> [大丙](https://subingwen.cn/linux/vim/)
>
> 安装
>
> https://github.com/youngyangyang04/PowerVim/
>
> 安装的这个，不确定装在用户里还是系统里



> 处理 /home/wsy/.vimrc 时发生错误: 
> 第 42 行: 
> E197: 不能设定语言为 "zh_CN.gb2312" 
>
> > 应该是taglist.vim插件的问题：在~/.vimrc 文件中加入如下三行就可以了 
> > let Tlist_Show_One_File=1     "不同时显示多个文件的tag，只显示当前文件的 
> > let Tlist_Exit_OnlyWindow=1   "如果taglist窗口是最后一个窗口，则退出vim 
> > let Tlist_Ctags_Cmd="/usr/bin/ctags" "将taglist与ctags关联
> >
> > 试验了下这个方法不行 
>
> 在终端中通过locale命令查看了下，发现好像确实没有gb2312编码，然后就打算用下面的方法安装中文环境
>
> [ubuntu安装中文环境 zh_CN.GB2312 zh_CN.GBK](https://blog.csdn.net/hm_123123123/article/details/107076489?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2.pc_relevant_paycolumn_v3) 这个方法没找到那个文件，而且没权限保存



![image-20220411103146553](E:\MarkDown\picture\image-20220411103146553.png)

[vim使用(已下载，存在 基础学习/vim)](https://github.com/wsdjeg/vim-galore-zh_cn)

Vim 自带一个交互式的教程，内含你需要了解的最基础的信息，你可以通过终端运行以下命令打开教程：

    vimtutor



> 整体流程
>
> mkdir c++
> cd c++
> torch main.cpp 创建文件
> vi main.cpp 进入文件
>
> vim main.cpp 创建并进入
>
> :wq 保存并退出



vim xxx，进入正常模式，此时是无法输入的；

hjkl分别为左下上右，x为删除光标下的字符。

想要修改的话，比如mising，想要在i前加一个s，需要把光标移动到i上，然后点**i进入编辑模式**，输入要添加的字符，然后ESC进入正常模式，再移动光标进行其他的修改

按a键，自动跳到光标的末尾，然后进入编辑模式。与i的区别是一个是Insert，一个是在末尾append

:q!为直接退出，:wq可以保存文件并退出，:w保存，:w!强制保存





dw，与x删除一个字母不同，dw是删除一个单词，包括单词后面的空格，d2w就是删除两个空格，de是仅删除那个单词。d$是删除光标后的一整个句子，即删除光标及光标后的一整行

u撤销上一条命令，U是恢复该行所有的更改，ctrl+r撤销撤销的命令

v 然后按方向键，进行文本的选中
y 复制选中的内容，yy 复制一行
d 删除选中的内容，dd：删除一整行，2dd删除两行
p：paste，粘贴。将刚删除的一个单词或一个字母、一行文字等放在光标后面，句子的话放在光标下面那行。P粘贴在光标前，句子是光标上方。

w：移动到下个单词的第一个字母，b则是移动到上个(当前)单词的第一个字母
e：移动到单词的最后一个字母
0：跳转到一行的第一个字母

gg 跳转到文件首
G 跳转到文件末尾
ggdG 剪贴文件所有内容

/test 斜杠加字母，就是查找某个字符串，比如这里查找test

:e file2 命令加载另一个文件

**看到3.2了**

![image-20220411103454838](E:\MarkDown\picture\image-20220411103454838.png)



# gdb

gdb可以对可执行文件进行调试，在编译时需要打开调试选项(-g)

```shell
# -g 将调试信息写入到可执行程序中
gcc -g args.c -o app

# 正常编译时不添加 -g 参数
gcc args.c -o app1  

# 查看生成的两个可执行程序的大小
ll # 相当于 ls -l

#################################################################
-rwxrwxr-x  1 robin robin 9816 Apr 19 09:25 app*	# 可以用于gdb调试
-rwxrwxr-x  1 robin robin 8608 Apr 19 09:25 app1*	# 不能用于gdb调试
```



```shell
# 启动gdb
gdb app

# 如果是 int main(int argc, char* argv[]) 这种形式，需要传参的话：
# 设置的时机: 启动gdb之后, 在应用程序启动之前
set args 参数1 参数2 .... ...
# 查看设置的命令行参数
show args


# gdb 中启动程序，可以使用run或start
# run: 可以缩写为 r, 如果程序中设置了断点会停在第一个断点的位置，如果没有设置断点，程序就执行完了
# start: 启动程序，最终会阻塞在 main 函数的第一行，等待输入后续其它 gdb 指令
run
start

# 如果想让程序 start 之后继续运行，或者在断点处继续运行，可以使用 continue 命令，可以简写为 c
continue


# 设置断点：break / b
# 在当前文件的某一行上设置断点
b 行号
b 函数名		# 停止在函数的第一行
# 在非当前文件的某一行上设置断点
b 文件名:行号
b 文件名:函数名		# 停止在函数的第一行

# 删除断点
# delete == del == d
# 需要 info b 查看断点的信息, 第一列就是编号
d 断点的编号1 [断点编号2 ...]
# 举例: 
d 1          # 删除第1个断点
d 2 4 6      # 删除第2,4,6个断点

# 删除一个范围, 断点编号 num1 - numN 是一个连续区间
(gdb) d num1-numN
# 举例, 删除第1到第5个断点
(gdb) d 1-5

# 打印变量名
print 变量名
p 变量名

# 在断点上单步调试
# 命令被执行一次代码被向下执行一行，如果这一行是一个函数调用，那么程序会进入到函数体内部。
step
s

# 设置变量值
# 可以在循环中使用, 直接设置循环因子的值
# 假设某个变量的值在程序中==90的概率是5%, 这时候可以直接通过命令将这个变量值设置为90
set var 变量名=值

# 退出gdb
quit
q
```







使用gdb attach PID可以调试运行中的程序，如a.out进程，命令如下：

gdb attach 338821

进入gdb调试命令行以后，打印所有线程的调用栈信息，信息如下：
thread apply all bt命令查看所用线程堆栈信息

![image-20230411162959010](E:\MarkDown\picture\image-20230411162959010.png)
![image-20230411163033066](E:\MarkDown\picture\image-20230411163033066.png)

从上面的线程调用栈信息可以看到，当前进程有三个线程，分别是**Thread1是main线程，Thread2是taskA线程，Thread3是taskB线程**。

从调用栈信息可以看到，Thread3线程进入S阻塞状态的原因是因为它最后在#0 __lll_lock_wait () at，也就是它在等待获取一把锁(lock_wait)，而且堆栈信息打印的很清晰，#1  0x00007f9ca3cffb09 in pthread_mutex_lock () from /lib64/libpthread.so.0，Thread3在获取而获取不到，因此进入阻塞状态了。
Thread2同理。

# linux命令

> reference:
>
> https://mp.weixin.qq.com/s/JJM6eZMUKfgpfU2Kjj__Sw
>
> 大丙https://subingwen.cn/linux/

cd、ssh、yum(管理软件包，因为云服务器用的是centos系统)、kill（结束一个进程）

## ifconfig

查看 `ip` 网络相关信息

## man

查看帮助信息，man + 数字 + 命令/函数，如`man 7 tcp`

1. 可执行程序或 `Shell` 命令；
2. 系统调用（ `Linux` 内核提供的函数）；
3. 库调用（程序库中的函数）；
4. 文件（例如 `/etc/passwd` ）；
5. 特殊文件（通常在 `/dev` 下）；
6. 游戏；
7. 杂项（ `man(7)` ，`groff(7)` ）；
8. 系统管理命令（通常只能被 `root` 用户使用）；
9. 内核子程序。



## chown

设置文件所有者和文件关联组的命令

* -R : 处理指定目录以及其子目录下的所有文件

```shell
# 将文件 file1.txt 的拥有者设为 www，群体的使用者 root :
chown www:www file1.txt
```



## chmod

> 777，修改权限命令是啥 

linux中root是超级用户，拥有最高权限；sudo命令可以以root身份运行命令

**chmod**

修改文件访问权限

【常用参数】

- `-R` 可以递归地修改文件访问权限，例如 `chmod -R 777 /home/lion`

**文件权限**

```
// ls -l 查看文件详细信息，这里先显示一个
drwxr-xr-x 5 root root 4096 Apr 13  2020 climb
```

- `d` ：表示目录，就是说这是一个目录，普通文件是 `-` ，链接是 `l` 。
- `r` ：`read` 表示文件可读。
- `w` ：`write` 表示文件可写，一般有写的权限，就有删除的权限。
- `x` ：`execute` 表示文件可执行。
- `-` ：表示没有相应权限。

drwxr-xr-x要分为四段，即 d rwx r-x r-x，分别对应文件属性、所有者、群组用户、其他用户

- d:是一个文件夹；
- rwx:所有者具有：读、写、执行权限；
- r-x:群组用户具有：读、执行的权限，没有写的权限；
- r-x:其它用户具有：读、执行的权限，没有写的权限。

**数字分配权限**

| 权限 | 数字 |
| :--: | :--: |
|  r   |  4   |
|  w   |  2   |
|  x   |  1   |

```shell
chmod 640 hello.c
```

* 6 = 4 + 2 + 0 表示所有者具有 rw 权限
* 4 = 4 + 0 + 0 表示群组用户具有 r 权限
* 0 = 0 + 0 + 0 表示其它用户没有权限
* 777就代表所有者、群组用户和其他用户都可读可写可执行

**字母来分配权限**

```shell
chmod u+rx file --> 文件file的所有者增加读和运行的权限
chmod g+r file --> 文件file的群组用户增加读的权限
chmod o-r file --> 文件file的其它用户移除读的权限
chmod g+r o-r file --> 文件file的群组用户增加读的权限，其它用户移除读的权限
chmod go-r file --> 文件file的群组和其他用户移除读的权限
chmod +x file --> 文件file的所有用户增加运行的权限
chmod u=rwx,g=r,o=- file --> 文件file的所有者分配读写和执行的权限，群组其它用户分配读的权限，其他用户没有任何权限
```

- `u` ：`user` 的缩写，用户的意思，表示所有者。
- `g` ：`group` 的缩写，群组的意思，表示群组用户。
- `o` ：`other` 的缩写，其它的意思，表示其它用户。
- `a` ：`all` 的缩写，所有的意思，表示所有用户。
- `+` ：加号，表示添加权限。
- `-` ：减号，表示去除权限。
- `=` ：等于号，表示分配权限。



## wget

下载文件

```shell
wget http://www.minjieren.com/wordpress-3.1-zh_CN.zip
wget -c xxx # 断续下载
```



wget 参[URL地址]

## 查找命令

### find

```shell
# 语法格式: 根据文件名搜索
#  * 可以匹配零个或者多个字符, ?用于匹配单个字符。
find 搜索的路径 -name 要搜索的文件名
find /root -name "*.txt"

# 查找文件夹，其中第一个/为查找范围，d为目录，inames忽略字符大小写的差别；
find / -type d -iname "ssh"

# 将 find 搜索的结果通过管道传递给后边的shell命令继续处理
find 路径 参数 参数值 | xargs shell命令2
# 查找文件, 并且显示文件的详细信息
find ./ -maxdepth 1  -name "*.cpp" | xargs ls -l
```



### grep

​	用于查找文件里符合条件的字符串或正则表达式。

`grep [options] pattern [files]`

* pattern - 表示要查找的字符串或正则表达式。
* files - 表示要查找的文件名，可以同时查找多个文件，如果省略 files 参数，则默认从标准输入中读取数据。
* 常用 options
  * `-i`：忽略大小写进行匹配。
  * `-n`：显示匹配行的行号。
  * `-r`：递归查找子目录中的文件。
  * `-l`：只打印匹配的文件名。
  * `-c`：只打印匹配的行数
  * `-v`：反向查找，只打印不匹配的行。

```shell
// 在文件 file.txt 中查找字符串 "hello"，并打印匹配的行：
grep hello file.txt -n
```



## ==pwd==

显示当前目录的路径

pwd -P 显示出实际路径

## tar

```shell
# 将archive文件夹归档并压缩
tar -zcvf archive.tar.gz archive/ 
# 将archive.tar.gz归档压缩文件解压
tar -zxvf archive.tar.gz 
```

* z: 使用 gzip 的方式进行文件压缩
* `-cvf` 表示 `create`（创建）+ `verbose`（细节）+ `file`（文件），创建归档文件并显示操作细节；(压缩)
* `-xvf` 解开归档（解压），`tar -xvf archive.tar`

## zip

```shell
# 因为 centos 可以使用 root 用户登录, 基于 root 用户安装软件, 不需要加 sudo
sudo yum install zip   # 压缩
sudo yum install unzip # 解压缩
```

压缩和解压缩就是zip和unzip命令

## cd

切换到某个目录

cd /d

特殊的目录:
..: 表示当前目录的上一级目录，使用 cd .. 或者 cd ../ 都可以
. : 表示当前目录，使用 . 或者./ 都可以，cd . 不会切换目录

## ls

列出文件和目录，

* `-a` 显示所有文件和目录包括**隐藏**的
* `-l` 显示详细列表，`ls -l`这个命令比较常用，显式详细信息，缩写为`ll`
* `-h` 适合人类阅读的
* `-t` 按文件最近一次修改时间排序
* `-i` 显示文件的 `inode` （ `inode` 是文件内容的标识）



## ln

ln [选项] 源文件 目标文件

选项：

* -s：建立软链接文件。如果不加 "-s" 选项，则建立硬链接文件；
* -f：强制。如果目标文件已经存在，则删除目标文件后再建立链接文件；
* 注意，**软链接**文件的源文件必须写成**绝对路径**，而不能写成相对路径（硬链接没有这样的要求），否则软链接文件会报错

​	每当我们给给磁盘文件创建一个硬链接（使用 ln），磁盘上就会出现一个新的文件名，硬链接计数加 1，但是这新文件并不占用任何的磁盘空间，文件名还是映射到原来的磁盘地址上。与copy是不同的
<img src="E:\MarkDown\picture\image-20230619145506930.png" alt="image-20230619145506930" style="zoom:50%;" />



## cat

一次性显示文件所有内容，更适合查看小的文件。

```shell
cat cloud-init.log
```

`-n` 显示行号。

搭配grep使用，查找文件中内容

```cat a.txt | grep cc
cat a.txt | grep cc
```



## less

分页显示文件内容，更适合查看大的文件。

* 空格键：前进一页（一个屏幕）；
* `b` 键：后退一页（向上翻页）；
* 回车：显式下一行；
* q：退出

## touch

创建一个文件`touch test.cpp`

## cp

拷贝文件和目录

```shell
cp a.cpp test # 将a.cpp移动到test目录下
cp 目录A 目录B -r # 拷贝目录需要加参数-r
```



## mv

移动或重命名文件或目录，与cp命令用法相似。

## rm

删除文件和目录

rm -rf xxx 删除xxx文件夹下所有东西

## mkdir

创建目录

```shell
# 单层目录
$ mkdir 新目录的名字

# 多层目录, 需要加参数 -p
$ mkdir parent/child/baby1/baby2 -p
```

## 重定向

```shell
# 将date数据放到txt文件里
date > test.txt
```



* 执行一个shell指令, 获得一个输出, 这个输出默认显示到终端, 如果要将其保存到文件中, 就可以使用重定向

* `>`: 将文件内容写入到指定文件中，如果文件中已有数据，则会使用新数据覆盖原数据
* `>>`: 将输出的内容追加到指定的文件尾部

## 管道 `|`

把两个命令连起来使用，一个命令的输出作为另外一个命令的输入，英文是 `pipeline` ，可以想象一个个水管连接起来，管道算是重定向流的一种。

## kill

kill PID

kill -9 PID 是强制结束进程

## ==查看CPU或者内存信息==

## 进程状态

1. 状态码 `R` ：表示正在运行的状态；
2. 状态码 `S` ：表示中断（休眠中，受阻，当某个条件形成后或接受到信号时，则脱离该状态）；
3. 状态码 `D` ：表示不可中断（进程不响应系统异步信号，即使用kill命令也不能使其中断）；
4. 状态码 `Z` ：表示僵死（进程已终止，但进程描述符依然存在，直到父进程调用 `wait4()` 系统函数后将进程释放）；
5. 状态码 `T` ：表示停止（进程收到 `SIGSTOP` 、 `SIGSTP` 、 `SIGTIN` 、 `SIGTOU` 等停止信号后停止运行）。

## free

free命令用于显示内存使用信息。显示Memory和Swap

```shell
free # 显示内存使用信息
free -t # 以总和的形式查询内存的使用信息
free -s 10 # 每10s 执行一次命令,周期性的查询内存使用信息
```

* -b 以Byte为单位显示内存使用情况。
* -k 以KB为单位显示内存使用情况。
* -m 以MB为单位显示内存使用情况。

## ps

用于显示当前系统中的进程，显示的是静态的进程列表

`-ef` 列出所有进程;

`-efH` 以乔木状列举出所有进程;

`-u` 列出此用户运行的进程;

`-aux` 通过 `CPU` 和内存使用来过滤进程 `ps -aux | less` ，显示PID、CPU占用、内存占用、进程状态。

`-aux --sort -pcpu` 按 `CPU` 使用降序排列， `-aux --sort -pmem` 表示按内存使用降序排列;

```shell
ps -aux | grep a.out # 在进程中查询有a.outz
ps -aux --sort -pcpu # 按 `CPU` 使用降序排列
ps -aux --sort -pmem # 按内存使用降序排列;
```

![image-20230411185054966](D:\MarkDown\picture\image-20230411185054966.png)

Sl+ : S 处于休眠状态、l 多线程、+ 位于后台的进程组；

## top

获取进程的**动态**列表。

进入top页面后，按`P`是按照CPU占用排序显示，`M`是按内存占用排序显示。

| H    | 显示所有线程的运行状态指标。如果没有该参数，会显示一个进程中所有线程的总和。在运行过程中，可以通过H命令进行交互控制。 |
| ---- | ------------------------------------------------------------ |
| p    | 通过指定监控进程ID来仅仅监控某个进程的状态。可以指定多个，-pN1 -pN2 ...  或者 -pN1,N2,N3 ... |

```cpp
top -Hp 338821
```

top命令也能看到当前的系统的软中断情况，其中黄色部分 `si`，就是 CPU 在软中断上的使用率

<img src="E:\MarkDown\picture\image-20230727213151619.png" alt="image-20230727213151619" style="zoom:60%;" />

## shell的命令

### shell脚本

xxx.sh

```shell
#!/bin/bash
# 第一行为固定格式
echo "Hello World !"
```



### echo

用于字符串的输出，`echo string`
