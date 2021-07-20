# Github使用手册

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

- 查看文件变化状态

  `$git status`

- 查看文件变化内容

  `$git diff 文件.txt`

- 每次提交修改都需要add、commit

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



##### github

- 连接github

  1. 创建SSH key

     `$ ssh-keygen -t rsa -C "youremail@example.com"`

     - 在用户目录下的.ssh目录中会出现两个文件
     - id_rsa是私钥，注意防泄漏
     - id_rsa.pub是公钥，可公开

  2. 在github用户设置中打开SSH Keys，粘贴公钥

  3. 点add key
  
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

    

- 克隆到一个本地库

  `$git clone git@github.com:用户/库.git`



##### 分支管理

- 创建分支

  `$git switch -c 分支名 ` 或

  `$git checkout -b 分支名`

  - -b （-c）创建并切换，相当于以下两条命令
    - `$git branch 分支名 `
    - `$git checkout 分支名`

- 查看当前分支(当前分支前会有一个*)

  `$git branch`
  
- 合并分支

  `$git merge 最新的分支名`

  - 产生冲突时需手动解决冲突

- 删除分支

  `$git branch -d 分支名`













