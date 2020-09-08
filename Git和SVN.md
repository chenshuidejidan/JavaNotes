# SVN和Git
- SVN（CVCS）是集中式管理的版本控制器，而Git(DVCS) 是分布式管理的版本控制器！这是两者之间最核心的区别
## 1. SVN属于集中式的版本控制系统

SVN原理上只关心文件内容的具体差异。每次记录有哪些文件作了更新，以及都更新了哪些行的什么内容
### 优点：
- 管理方便，逻辑明确，符合一般人思维习惯。
- 易于管理，集中式服务器更能保证安全性。
- 代码一致性非常高。
- 适合开发人数不多的项目开发。

### 缺点：
- 服务器压力太大，数据库容量暴增。
- 如果不能连接到服务器上，基本上不可以工作，看上面第二步，如果服务器不能连接上，就不能提交，还原，对比等等。
- 不适合开源开发（开发人数非常非常多，但是Google app engine就是用svn的）。但是一般集中式管理的有非常明确的权限管理机制（例如分支访问限制），可以实现分层管理，从而很好的解决开发人数众多的问题。

## 2. Git属于分布式的版本控制系统
### 优点：
- 适合分布式开发，强调个体。
- 公共服务器压力和数据量都不会太大。
- 速度快、灵活。
- 任意两个开发者之间可以很容易的解决冲突。
- 离线工作。
### 缺点：
- 学习周期相对而言比较长。
- 不符合常规思维。
- 代码保密性差，一旦开发者把整个库克隆下来就可以完全公开所有代码和版本信息。


# Git

## 1. 本地仓库命令
- 配置git：`git config --global user.name "sssss"`  
`git config --global user.email 123456@qq.com` 
- 创建本地仓库： `git init`
- 查看本地仓库状态：`git status`
- 将文件添加到暂存区：`git add file`，可以使用`git add .` 将所有修改的文件进行添加
- 删除暂存区文件：`git rm file`
- 提交到本地仓库：`git commit -m “提交描述”`
- 将所有修改提交：`git commit -am “提交描述”`，将工作区所有文件添加到暂存区并提交
- 撤销commit：`git reset --soft HEAD^`，`HEAD^`等价于`HEAD~1`，两次commit都想撤销可以使用`HEAD~2`
  - 可选参数：
  - --mixed：不删除工作空间改动代码，撤销commit，并且撤销 git add 操作，为默认参数
  - --soft： 不删除工作空间改动代码，撤销commit，不撤销git add 操作
  - --hard： 删除工作空间改动代码，撤销commit，撤销git add ，恢复工作空间到上一次commit的状态
- 克隆仓库：`git clone git@github.com:xxxx.git`，完整克隆，可以直接看到修改记录


## 2. 本地仓库推送到远程ssh
- ssh方式使用密钥认证，不用输入账号密码了
- 生成ssh密钥：`ssh-keygen -t rsa`，默认生成位置为：`C:\Users\xxx\.ssh`，将公钥复制到github个人的SSH and GPG keys
- 建立连接：`git remote add origin git@github.com:xxxx.git`
- pull：`git pull --rebase origin master`
- push：`git push -u origin master`

## 3. 本地仓库推送到远程https
- 建立连接：`git remote add origin https://github.com/xxxx.git`
- pull：`git pull --rebase origin master`
- push：`git push -u origin master`

# SVN
