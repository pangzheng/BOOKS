# 学习 GIT

## 技巧
- 如果 commit 提交错误了想修改，通过以下命令进入编辑部分，然后重新 push
	- 当前正在提交的版本
		
			git commit --amend
	- 修改原来有 commit id 的版本

			git rebase -i commit id
- 撤销 commit 
	- 找到上次git commit的 id

			git log 
	- 同时将代码恢复到 commit_id 对应的版本
   			
   			git reset --hard commit_id
	- 只回退 commit 不回退代码
	
			git reset commit_id 			
## 分支开发
- 创建分支
- 分支提交代码到 master
	- 切换回 master 分支
	
			git checkout master
	- 获取最新 master 代码
	
			git pull --rebase
	- 切换到自己的分支
		
			git checkout zpang
	- 同步主分支(注意这里如果已经修改需要 commit)

			git rebase master
	- 查看分支情况

			git status
	- 开发和提交
	
			git add .
			git commit -m "xx" 		
	- 提交代码

			git push
	- git 页面操作
		- 切换到自己分支点击 new pull request
		- 查看 pr 代码并切合并					

## rebase 冲突如何处理
- 当主分支因为意外需要回滚，而你的分支已经更新了，就会受到回滚的影响那么就需要一下操作
	- 先回到主分支进行更新，获取最新的主分支
	
			git checkout master
			git pull --rebase
	- 切换到自己的分支，并同步主分支，并且提示有一个提交

			git checkout zpang		
			git rebase master
		- 如果想合并之前分支提交在主线上只提交一次，那么使用
			
				git rebase -i master
				
			- 进入选择后，保留一个 `pick` ,其他的都改成 `s` ,然后记录
			- 进入 commit 后，删除无用的 commit 只保留一个			
		- 如果有冲突，查看提交并解决

				git status
		- 编辑冲突的文件,找到冲突点，手动解决冲突

				vi xxxx
		- 将编辑文件重新加入

				git add xxx
		- 因为这些错误还是在 rebase 过程中，所以需要继续rebase 操作

				git rebase --continue
		- 是否还有报错，如果有返回这个流程重新处理
		- 提交无冲突代码到自己的主线分支

				git push -f 			
		- 测试没问题后到 `github` 上提交 `pr`

## fork 同步
- 到 fork 的项目目录添加一个新的分支，并将原项目获取

		git remote add upstream https://github.com/whoever/whatever.git
- fetch 这个项目

		git fetch upstream
- 切换到 master 分支

		git checkout master
- 合并

		git rebase upstream/master	
- 提交

		git push
		git push --tags		
					
## 资料
[learn-git](https://www.atlassian.com/git/tutorials/learn-git-with-bitbucket-cloud)