## basic use

git config --global user.name "username"  
git config --global user.email "email"
git clone ssh://i520726@git.wdf.sap.corp:29418/hanalite-releasepack 
git status
git branch 
git fetch
git rebase origin/im_main
git branch +branchname 新建分支
git branch -d +branchname 删除分支
git checkout +branchname 切换分支
git checkout -b +branchname 创建并切换分支
git add a.txt，使a.txt变成已跟踪状态
git commit -a -m +coment 提交所有修改
git merge +branchname 将branchname分支合并到当前分支
git branch --merged 查看哪些分支已经合并到当前分支
git branch -v 查看每一个分支的最后一次提交
git reset –soft <版本号>重置至指定版本的提交,参数hard，会撤销相应工作区的修改
git commit --amend 合并缓存的修改和上一次的提交，用新的快照替换上一个提交。缓存区没有文件时运行这个命令可以用来编辑上次提交的提交信息，而不会更改快照
git push的一般形式为 git push <远程主机名> <本地分支名>  <远程分支名> ，例如 git push origin master：refs/for/master ，即是将本地的master分支推送到远程主机origin上的对应master分支， origin 是远程主机名,refs/for 的意义在于我们提交代码到服务器之后是需要经过code review 之后才能进行merge的，而refs/heads 不需要
git remote add origin https://git.oschina.net/name/package.git  // 远程仓库名+远程仓库地址 创建远程仓库
git remote -v
git remote add upstream https://github.wdf.sap.corp/bdh/bdh-infra-tools.git
git fetch upstream
git merge upstream/master        将从远程仓库upstream上fetch下来的master分支的代码合并到当前master分支
git push origin master:master    将本地仓库master分支上的代码push到远程origin仓库的master分支下
提交
git add
git commit -m ""
git push origin master

cherk-pick步骤：
git checkout stable
git cherry-pick 18c0392  这个数字是在master分支上的
git push
到i-number的git 上，branch改成stable，create new PR，
不能加reviewer，只能在slack里发信，@上max， Edward。他们会看。
git checkout rel-3.0
git cherry-pick 18c0392
git push


git修改远程仓库地址 

方法有三种：

1.修改命令

git remote set-url origin [url]

2.先删后加

git remote rm origin
git remote add origin [url]

3.直接修改config文件


查看最近3次提交记录
git log --oneline -3 

rel-dhaas: https://github.wdf.sap.corp/bdh/bdh-infra-tools/pull/3844



## windows：

error: filename in tree entry contains backslash: '\'
git config --global core.protectNTFS false

给文件加权限
git ls-files --stage |grep vctl_import_ca.sh
git update-index --chmod=+x common/ansible/roles/vora/vora-kubernetes-install/files/vctl_import_ca.sh



## gerrit change

	git checkout stable
	git pull
	git checkout -b my944
	git rebase stable
	git git push origin HEAD:refs/for/stable

## update submodule

git submodule update --init
git submodule update --remote -- bdh-infra-tools
git add bdh-infra-tools


git fetch upstream pull/4472/head:pull4472



gerrit ammend
	edit xxx
	git add xxx
	git commit --amend --no-edit
	git push origin @:refs/for/master
	
配置代理
	
git config --global http.proxy http://127.0.0.1:7890

git config --global https.proxy https://127.0.0.1:7890

删除代理

git config --global --unset http.proxy 

git config --global --unset https.proxy 



配置globle 环境变量

git config --global --edit







首先是到yum源设置文件夹里: /etc/yum.repos.d/
1. 查看yum源信息:
    yum repolist
2. 安装base reop源
     cd /etc/yum.repos.d
3. 接着备份旧的配置文件
   sudo mv CentOS-Base.repo CentOS-Base.repo.bak
4. 下载阿里源的文件
    sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
    5.清理缓存
    yum clean all
    6.重新生成缓存
    yum makecache
7. 再次查看yum源信息
   yum repolist
 8. 安装
      yum -y update 
