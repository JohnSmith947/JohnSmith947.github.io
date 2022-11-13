---

title: Git 相关笔记整理

date: 2022-11-13 20:05:10

tags:

---
# Git 相关笔记整理

前言：

​	整理了日常工作开发中常用的 Git 相关知识，不断完善。



## 一、提交规范

​	Git 每次提交代码都需要写 commit message，否则就不允许提交，直接说明了 commit message 的重要性。commit message 应该说明白本次提交的目的以及作了何种操作。

**commit message 格式**

```java
<type>(<scope>):<subject>
```

- **type：用于说明 git commit 的类别，只允许使用下面的标识：**
  - feat：新功能（feature）。
  - fix/to：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。
  - fix：产生diff并自动修复此问题。适合于一次提交直接修复问题
  - to：只产生diff不自动修复此问题。适合于多次提交。最终修复问题提交时使用fix
  - docs：文档（documentation）。
  - style：格式（不影响代码运行的变动）。
  - refactor：重构（即不是新增功能，也不是修改bug的代码变动）。
  - perf：优化相关，比如提升性能、体验。
  - test：增加测试。
  - chore：构建过程或辅助工具的变动。
  - revert：回滚到上一个版本。
  - merge：代码合并。
  - sync：同步主线或分支的Bug。



- **scope（可选）：scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。也可以写当前需求的名称。**



- **subject(必须)：subject是commit目的的简短描述，不超过50个字符。**
  - 建议使用中文
  - 结尾不加句号或其他标点符号



- **示例：**
  - feat(员工管理系统):增加数据层接口
  - fix(控制层):修复日志输出bug





## 二、代码回滚

### 用 IDEA 回滚代码

#### 第一种方式

- 当前的两次提交：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b598a0909cf4a429254f261a46a4998~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 在 git log 里，复制 `当前版本（提交错误的版本）` 及 `目标版本` 的版本号，并记录下来，如：

> 当前版本：d6c98d607b05127f80de8d5b360cf7f588df9c0e
>
> 目标版本：baba3e15ea9ebdcc50629a42791385591af4ff3c

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac4cb42c50444c3a99f6d346961130e9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 右击项目或者在菜单栏点击 Git 菜单，点击 Reset HEAD...

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/235d8571576846298af4e02da18f0b0a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 在弹出的输入框内选择 Reset Type 为 `Hard`，To  Commit 输入框输入`目标版本`的版本号：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a7ff174911d4e91b0e7b469be9789a0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 点击 Reset 后，代码会回到目标版本的样子，此时无法正常 push 到远程，如果强推的话，有些公司并不允许。所以应该再作一次如上操作：右击项目或者在菜单栏点击 Git 菜单，点击 Reset HEAD...，在弹出的输入框内选择 Reset Type 为 `Mixed`，To  Commit 输入框输入 `当前版本` 的版本号：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a5725c6c6d84302b13e14f3f9cf13b2~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 点击 Reset 后，发现待提交有了文件并且可以 commit 和 push 了，虽然会留下记录，但是达到了回滚代码的目的。



#### 第二种方式

- 当前的两次提交：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1a4310a74ba4aaa81ef1943635bffdb~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 目的：回到“最初的”这个版本。在 git 中右击“最初的”这个版本，选择图上所示选项：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1449f7fd4e5c4ba49e68910b307676c8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 出现如下对话框：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f06ff9d007e142b1afcfabad851d4526~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 简单来理解，选择 Soft 或是 Mixed，只是将指针指向了目标提交，错误提交的代码仍然存在，但是在 IDEA 中，文件会以蓝色标识展示，代码则以绿色标识展示，这样可以比较下你这两次提交有哪些地方作了修改。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e35d24a137964318afe51f6e90081a37~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 选择 Hard 或是 Keep，你的代码将会回退到你选中的那个版本的样子，没有任何标识。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5695d98e0fb24dc3bc7cdc610ec37ac5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 不管是四个选项中的哪一个，如果你的代码跟目标版本完全一致，将无法进行 push。关于这四个选项将在下面展开说。



#### 第三种方式

- 当前的数次提交如下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7432596d0b14380a45c618bc3059fef~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 假如第4次提交为错误提交，则在 git log 中右击该提交，选择 Revert Commit

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38e831703cde495aa8f7d1d3380f9c08~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 直接撤销了这一次提交的代码，也会生成一条新的 commit，然后可直接 push 到远程仓库。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/707a10836b5e45f0a25f287dba1bf6b9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 另一种情况：已提交但是未 push 到远程仓库的情况下，如图所示，“来一下子”为错误提交

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1aa21e7719e4464f83656a61233851c6~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 直接右击该提交，选择 Undo Commit

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90a56439624b4215a9147d1f015e0e24~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 点击 OK

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14eceef07c274351959e033785e1ff79~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 你会发现上一次提交被移出了版本区，在 IDEA 中可以查看到该次提交所做的改变，方便对照。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5163020db2db46f1843e45c23b95a06d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)



### 用命令行回滚代码

#### 第一种方式

```shell
# 查看 git 提交的历史记录
git reflog
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1560c37041a64721a9cac1ffe61d4b53~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

```shell
# 回退版本，commit_id 为 log 中最前面的数字字母组合
git revert -n <commit_id>
```

**这里分两种情况：**

- **撤销上一次提交**

```shell
# 以上图为例，这里的 commit_id 为 HEAD 指向的当前版本
git revert -n 978bc9f
```

这里类似 IDEA 回滚代码的第三种方式中的 Undo Commit一样的，后面修改完直接 push 就好。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d7c19124f624e79901e74b81612e64d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- **回退到某个版本**

```shell
# 还是图为例，这里的 commit_id 为 “123”的版本（在上图的基础上我又提交了好几次，所以123并不算上一次）
git revert -n f18bcd8
```

​	可能会出现如下情况，文件发生冲突，=========上面的为当前版本的内容，下面为目标版本的内容，但是发现下面的内容并不是想要的“123”。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c37a9ef306a246d2a4d0097543530286~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

​	换个视角来看，在 IDEA 的解决冲突窗口中，左边为当前分支内容，中间是目标版本的内容，右边则是目标分支的上一个版本的内容。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1011f2b9954404283a043f7c00bcecd~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

​	**结论：**由此可见，如果不通过看 IDEA 的解决冲突窗口来看，我们在命令行输入回退命令的时候应该选择“1234”那一条的 commit_id (978bc9f)，也就是目标版本的上一个版本，这样的话我们在文件中留下 ======= 下面的内容就可以了。

​	如果是通过 IDEA 的解决冲突窗口来看，则正常选择目标版本的 commit_id，然后直接点击 Apply 按钮以中间的内容为准即可。



#### 第二种方式

```shell
# HEAD 后面一个^表示上个版本，^^则表示上上个版本
git reset --hard HEAD^

# 回退到指定的版本，git reflog 中的 commit_id
git reset --hard <commit_id>

# n -> 数字，输入1则表示上一个版本
git reset --hard HEAD~n
```



### 单个文件回滚

​	在 IDEA 中可以使用 Git 的 History 功能来试试，下面展示用命令行方式回滚单个文件。

```shell
# 保留本地代码，把所有文件的提交记录都取消，回滚到指定版本
git reset --soft <commit_id>

# 从暂存区中把你需要回滚的文件移出去
git restore --staged ./README.md

# 直接提交（原理可参照第三点中“撤销暂存区的文件”）
git commit -m "message"
```





## 三、Q&A

#### IDEA 终端 git reflog 乱码

​	以下两种方式可解决，设置完毕后可能需要重启 IDE 或是电脑。

- **设置环境变量解决**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbcd1b9a4d1741dd9ed232eb5c58aa56~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- **临时解决**

```shell
# 直接终端中输入后回车即可
set LESSCHARSET=utf-8
```



#### Failed to connect to 127.0.0.1 port 7890 after 2020 ms: Connection refused

​	在 push 代码的时候出现该错误，拒绝连接。

```shell
# 查看是否使用代理
git config --global http.proxy

# 如果有的话则取消该代理
git config --global --unset http.proxy
```



#### OpenSSL SSL_connect: Connection was reset in connection to github.com:443

​	一般情况下是挂了 VPN 可能会出现的错误，还是代理的问题。首先我们可以选择重启一下 git 终端尝试一下，可能与当前的会话有关系，如果不行的话则使用如下方法：

```shell
# 设置一下代理端口号，假如端口号为 7890
git config --global http.proxy 127.0.0.1:7890
git config --global https.proxy 127.0.0.1:7890

# 如果已经有设置过代理，则输入以下命令取消后，再设置一次代理即可
git config --global --unset http.proxy
git config --global --unset https.proxy

# 查看 git 的 http 代理配置
git config --global http.proxy 
# 查看 git的 https 代理配置
git config --global https.proxy
# 查看 git 的所有配置
git config --global -l
```



#### 工作区、暂存区、版本区

- **工作区**

  编辑代码的目录文件夹。

- **暂存区**

  工作区中的文件使用 git add 命令后，就到了暂存区。

- **版本区**

  暂存区中的文件使用 git commit 即可添加到版本区。



#### git reflog 和 git log 的区别

- **git reflog**

  可以查看`所有分支`的所有操作记录信息（`包括`已经被删除的 commit 记录和 reset 的操作）。

  ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7624408f00624de79d62014f4fd0abda~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- **git log**

  可以显示`当前分支`所有提交过的版本信息，`不包括`已经被删除的 commit 记录和 reset 的操作。

  ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc4ace53f42843a8aa19de79d18e7cad~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

  直接使用 git log 显示内容过于繁琐，可以使用如下命令进行精简化显示。

  ```shell
  git log --pretty=oneline
  ```

  ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/833f557f41784b39809a9b246d2e02c2~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)



#### 基于原分支复制新的分支

```shell
# 切换到原分支
git checkout oldBranch
git pull

# 从原分支复制到新的分支
git checkout -b newBranch

# 将新的分支推送到远程仓库并关联
git push --set-upstream origin newbranch
git pull
```



#### Soft、Mixed、Hard、Keep的区别

​	回滚代码时的几个选项详解，建议先弄明白三大分区的概念。

- **Soft**
  - 移动本地库 HEAD 指针
- **Mixed**
  - 移动本地库 HEAD 指针
  - 重置暂存区
- **Hard**
  - 移动本地库 HEAD 指针
  - 重置暂存区
  - 重置工作区
- **Keep**
  - 移动本地库 HEAD 指针
  - 暂存区不变
  - 重置工作区



#### 删除本地分支

​	在 IDEA 中操作的话，直接点击该分支删除就好。

```shell
# 查看所有分支（不加 -a 仅查看本地分支），-a 是 --all 的简写
git branch -a

# 直接删除本地分支
git branch -d <branchName>

# 如果本地要删除的分支和上游的分支不一致，可能会报如下错误：
# error: The branch 'newbranch' is not fully merged.
# 使用如下命令进行删除，-D 是 --delete --force 的缩写，这样可以在不检查 merge 状态的情况下删除分支；--force 简写 -f，作用是将当前 branch 重置到初始点(startpoint)，如果不使用 --force 的话，git 分支无法修改一个已经存在的分支
git branch -D <branchName>
```



#### 删除远程分支

```shell
# 删除远程分支的时候你所在分支不能是要删除的那个分支，-d 是 --delete 的简写
git push origin -d branchName
```



#### 合并其他分支的某一次提交

- **IDEA 操作**

  当前是处于 test 分支，需要合并 newbranch 分支的“又一次新的提交”的内容，直接右击该提交选择 Cherry-pick 选项即可，有冲突则解决冲突。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df76df05eb28405bad53aaace3879459~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- **命令行操作**

```shell
# 查看所有分支的提交记录，找到要合并的那个分支的某一次提交的 commit_id
git reflog

# 使用 Cherry-pick 进行合并，有冲突则解决冲突
git cherry-pick <commit_id>
```



#### 暂存正在编辑中的代码

​	在当前分支进行开发时，正在编辑的文件还未提交至暂存区，这时需要切换到别的分支解决问题，可以使用 git stash 操作来进行缓存当前正在编辑中的内容。

- **IDEA 操作**

  - 当前未保存的内容

    ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60283479a28e4f6d905235745a3958b5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

  - 将未提交的内容缓存至栈中

  ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ad2f410f8654288853b37ad878d6437~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

  - 输入 message 便于识别，点击 Create Stash，会发现当前编辑的内容消失了，此时会保存在栈中便于随时进行恢复，现在切换到其他分支就不会影响进一步开发

  ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e12becf6d214c50a7ec7ca9a830c1e5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

  - 当在另一个分支开发结束，会到原来的分支时，需要将刚才缓存的内容恢复

  ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39981b67b0dc4f6f8070aaa858a167a8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

  - 在弹出的对话框中选择你刚才保存的那个，点击 Apply Stash 即可恢复刚才编辑的内容。可勾选 Pop stash 将这一次保存的内容删除掉

  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5e4c2e585f347d0b0a7ce85bd69b5f7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- **命令行操作**

```shell
# 将未提交的内容缓存
git stash save "message"

# 查看缓存的内容
git stash list

# 恢复内容，但不会删除保存记录，可以使用 git stash apply stash@{0} 来恢复指定记录
git stash apply

# 恢复内容，同时删除记录，可以使用 git stash pop stash@{0} 来恢复指定记录
git stash pop

# 删除最新一条记录，可以使用 git stash drop stash@{0} 来删除指定记录
git stash drop

# 清空保存记录
git stash clear
```



#### 合并其他分支的某一个文件

​	假如当前在 feature 分支，需要合并 dev 分支的 README.md 文件。

- **IDEA 操作**

  - 直接选择需要合并的文件右击，选择如下选项

  ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03cd3c36b11349888784c52f0f658107~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

  - 可以很直观的看到两个文件的差别，点击中间的箭头或者自行修改即可。

  ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e351eaa14705425f8e382c421842b512~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- **命令行操作**

```shell
# 这样做直接会把 dev 分支的 README.md 文件覆盖过来，后面的步骤就正常 add -> commit -> push 即可
# 如果是多个文件，可以在 .\README.md 后面继续写文件名，空格分隔
git checkout dev .\README.md
```



#### 分支的重命名

- **本地分支**

  IDEA 可以直接在分支切换处进行 Rename。

```shell
git branch -m <oldName> <newName>
```

- **远程分支**

```shell
# 修改本地分支名称
git branch -m <oldName> <newName>

# 删除远程分支
git push origin -d <oldName>

# 将修改好名字的分支推送到远程
git push --set-upstream origin newName
```



#### 撤销暂存区的文件

​	以下两个命令在 git status 命令下，会有提示。

- **git restore --staged <file>**

  ​	将文件从暂存区移出至工作区，且保留文件的最后一次修改。

  - 已经提交至暂存区

  ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76789a746411430996bbc150850ffe56~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

  - 使用 git restore --staged ./README.md 命令后，该文件将从暂存区移出，此时如果进行 commit 命令的话，该文件将不会提交至版本库。

  ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e818c72798ea4e6087c71b344f086a62~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- **git restore <file>**

  ​	已经在暂存区的文件进行了修改后，可以通过该命令，撤销本次修改，文件的内容将会改变。

  - 已经在暂存区的文件进行了修改的情况，图上的提示也很明显。

  ![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f75eb7d836ad4d7694308d05b1499f13~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

  - 此时如果使用 git restore ./README.md 命令，该文件将会撤销这次修改。



#### 强制推送

```shell
git push origin HEAD --force

git push -f
```



#### 查看分支与上游分支的关联情况

```shell
git branch -vv
```



