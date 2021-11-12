# Git使用

[TOC]

##### 安装配置

- 为自己起用户名与Email地址

  `$git config --global user.name"名字"`

  `$git config --global user.email "邮箱"`
  
  

##### 创建版本库

- 创建一个空目录（如果不存在则创建一个）

  `mkdir 目录名（📂）`

- 打开

  `cd 目录名`

- 显示当前目录

  `pwd`

- 把该目录变成Git可管理的仓库

  `git init`



##### 分支管理

- 创建分支

  `$git branch 分支名 `

- 切换分区

  `$git checkout 分支名 分支指向节点（若无，则指向HEAD）` 

  - 创建并切换（相当于以上两条命令）

    `$git checkout -b 分支名 要跟踪的远程分支（可选）`

- 查看所有分支(当前分支前会有一个*)

  `$git branch`

- 合并分支1：双父节点

  `$git merge 被合并分支名`

  - 当前所在分支移动到新生成的合并节点中，被合并分支仍在原位置
  - 新生成的合并节点有两个父节点，分别指向被合并的分支，和当前分支原来的位置
  - 对于相同的父节点，新生成的合并节点只会产生一次

- 合并分支2：单父节点

  `git rebase 被合并分支名`

  `git rebase 被合并分支名 要切换为当前分支的的分支名`

  - 合并节点的父节点只有被合并分支所指节点。当前分支指向合并节点，被合并分支指向不变
  - 合并的内容为当前节点以上直到共同父节点以下
  - 对于相同的父节点，新生成的合并节点只会产生一次。若被合并的分支指向的节点是当前分支的子节点，则当前分支指向被合并分支

- 复制节点到分支（类似合并的复制）1

  `git cherry-pick 节点1 节点2 ...`

  - 可以将其他分支节点放在当前分支之后（复制为子节点）
  - 当前分支会指向复制后的节点
  - 可以添加很多个节点一起复制

- 复制节点到分支（类似合并的剪切）2：交互式UI中完成

  `git rebase -i 继承链中产生分支的位置`

  - 通过拖动选择顺序、通过点击选择哪些节点不需要复制

- 删除分支

  `$git branch -d 分支名`



##### 版本控制

- .git目录（处于工作区）是git的版本库，内含暂存区(stage)
  
- 每个版本库有唯一的master分区
  
- 把文件添加到仓库（文件已经在该文件中）

  `$git add 文件.txt`

  - 这是将文件添加到暂存区
  - 添加所有文件：`$git add .`

- 提交

  `$git commit -m “写下这次的更新说明”`

  - 这是将暂存区的内容提交到分区
  - 移动的是当前所在branch

  `git commit --amend`

  - 发现上一次的提交有误（或者描述有误），使用此命令以覆盖上一次提交

- 查看文件变化状态

  `$git status`

- 查看文件变化内容

  `$git diff 文件.txt`

- HEAD：可以指向节点，也可以指向分支

- 切换到上n个节点（这里假设n=3）

  `git checkout master^^^`

  `git checkout master~3`

  - 只有HEAD指针指向了上n个节点，当前分支不变
  - 使用如`^3`（加数字）的方法用于指向父节点：如果有两个父节点，则会回到第n个父提交记录
  - 支持链式操作

- 强制修改分支指向位置

  `git branch -f 分支 节点`

- 撤销更改1：让最新的提交看起来像是不存在一样

  `git reset 上次提交的节点`

- 撤销更改2：新增子节点让他看起来像是上一个版本一样。适用于远程提交

  `git revert 要被撤销的节点`

  - 添加在当前节点之后

- 查看版本提交历史（从近到远）

  `$git log`

- 回退n个版本（回退后最新版本不会再出现再git log 中）

  `$git reset --hard HEAD~n`

- 取消回退并回到最新版本（使用commit id）

  `$git reset --hard commitId的前几位`

- 查看我的每一次命令》可以查看已删除的commit id

  `$git reflog`

- 撤销工作区修改（让文件回到最近一次commit或add）

  `$git checkout --文件.txt`

- 已add未commit时撤销修改

  `$git reset HEAD 文件.txt` +

  `$git checkout --文件.txt`

- 删除文件

  `$git rm 文件.txt` +

  `$git commit -m"消息"`

- 删后恢复

  `$git checkout --文件.txt`
  
- 加上标签

  `git tag 标签内容（如v1） 节点`

- 找到最近的标签

  `git describe 指定节点（空时表示HEAD）`

  - 输出结果为`<tag>_<numCommits>_g<hash>`
    - tag：离指定节点最近的标签
    - numCommits：与tag相差多少个提交记录
    - hash：指定节点的哈希值或标签（若有）



##### 远程仓库

- 连接github

  1. 创建SSH key

     `$ ssh-keygen -t rsa -C "youremail@example.com"`

     - 在用户目录下的.ssh目录中会出现两个文件
     - id_rsa是私钥，注意防泄漏
     - id_rsa.pub是公钥，可公开

  2. 在github用户设置中打开SSH Keys，粘贴公钥

  3. 点add key
  
- 远程分支的名字：`仓库名/分支名`

  - 仓库名默认为origin
  
- 远程仓库的HEAD与当前分支分离

- 远程仓库的拷贝（将git节点树完完整整的复制）

  `git clone git@github.com:用户/库.git`
  
- 获取远程仓库数据（更新节点树）

  `git fetch`

  - 这一操作仅仅是下载数据，不会对本地文件进行修改（只增不改）
  - 本地分支指向不会改变，但远程分支指向会得到更新

  `git fetch 仓库名 远程分支名`

  `git fetch 仓库名 远程节点：本地分支`

  - 参数中没有本地分支时，不移动本地分支
  - 本地分支指向远程节点对应本地仓库位置。若本地分支不存在，则直接创建
  - 远程节点为空时，会直接创建该分支

- 获取并合并为新节点

  `git pull`

  - 相对于fetch操作加上merge操作。默认为双父节点式
  - 添加参数` --rebase `更改为rebase式合并
  - 所以pull的参数和fetch参数一样，唯独本地分支不会指向合并节点而是拷贝的节点

- 上传节点至远程仓库

  `git push`

  - 远程分支也会指向最新上传的节点（指向所追踪的本地分支位置）
  - 倘若远程仓库的节点树和当前仓库不一样（指的只是不能够重合，允许本地仓库有多余的分支），则不能更新

  `git push 仓库名 同名的本地分支名`

  - 不改变当前所在分支地推送分支所在节点链

  `git push 仓库名 分支指向位置：远程分支名`

  - 分支指向的位置为空时，会删除掉该远程分支

- 与远程分支关联

  `git branch -u 远程分支名 创建的分支名（省略时表示当前分支）`

  `git checkout -b 创建的分支名 远程分支名`

  - 会在push或pull时改变对应指针位置
    - pull：本地分支跟随指向远程分支
    - push：远程分支跟随指向本地分支

- 添加远程库（在本地仓库下运行命令）

  ```
  git branch -M main
  git remote add origin https://github.com/rabbitAiry/appCreation.git
  git push -u origin main
  ```

- 提交到远程库（在本地仓库下运行命令）

  ```
  git config --global http.sslVerify "false"
  git push -u origin main
  ```

  - 提交时遇到错误OpenSSL SSL_read: Connection was reset, errno 10054。可以运行：

    ```
    git config --global http.sslVerify "false"
    ```

