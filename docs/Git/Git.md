# 命令

````sh
git init /*初始化*/
git checkout  -b main 	/* 更换分支 */
git remote add origin https://github.com/rainupup/rainupup.github.io.git	/* 指定上传 */

git add . 
git commit -m 'new'

/* 下拉 */
git pull origin main --allow-unrelated-histories		/* 当前文件夹下有内容 */
git pull origin main									/* 当前文件夹下没有内容 */
git push -u origin main									/* 上传 */
````

# 分支

````sh
# 列出所有本地分支
git branch

# 列出所有远程分支
git branch -r

# 新建一个分支，但依然停留在当前分支
git branch [branch-name]

# 新建一个分支，并切换到该分支
git checkout -b [branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
````



# 文件状态

**文件的四种状态**                                           

版本控制就是对文件的版本控制，要对文件进行修改、提交等操作，首先要知道文件当前在什么状态，不然可能会提交了现在还不想提交的文件，或者要提交的文件没提交上。

- Untracked: 未跟踪, 此文件在文件夹中,     但并没有加入到git库, 不参与版本控制. 通过git add 状态变为Staged.
- Unmodify: 文件已经入库, 未修改,     即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改, 而变为Modified. 如果使用git rm移出版本库, 则成为Untracked文件

- Modified: 文件已修改, 仅仅是修改,     并没有进行其他的操作. 这个文件也有两个去处, 通过git add可进入暂存staged状态, 使用git checkout 则丢弃修改过,    返回到unmodify状态, 这个git checkout即从库中取出文件, 覆盖当前修改 !

- Staged: 暂存状态. 执行git  commit则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为Unmodify状态. 执行git reset HEAD     filename取消暂存, 文件状态为Modified

**查看文件状态**  

上面说文件有4种状态，通过如下命令可以查看到文件的状态：

````sh
#查看指定文件状态
git status [filename]

#查看所有文件状态
git status
````

**忽略文件**                                               

有些时候我们不想把某些文件纳入版本控制中，比如数据库文件，临时文件，设计文件等

在主目录下建立".gitignore"文件，此文件有如下规则：

1. 忽略文件中的空行或以井号（#）开始的行将会被忽略。
2. 可以使用Linux通配符。例如：星号（*）代表任意多个字符，问号（？）代表一个字符，方括号（[abc]）代表可选字符范围，大括号（{string1,string2,...}）代表可选的字符串等。
3. 如果名称的最前面有一个感叹号（!），表示例外规则，将不被忽略。
4. 如果名称的最前面是一个路径分隔符（/），表示要忽略的文件在此目录下，而子目录中的文件不忽略。
5. 如果名称的最后面是一个路径分隔符（/），表示要忽略的是此目录下该名称的子目录，而非文件（默认文件或目录都忽略）。

````sh
#为注释
*.txt		#忽略所有 .txt结尾的文件,这样的话上传就不会被选中！
!lib.txt	#但lib.txt除外
/temp		#仅忽略项目根目录下的TODO文件,不包括其它目录temp
build/		#忽略build/目录下的所有文件
doc/*.txt	#会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
````





# 问题

`Failed to connect to github.com port 443 after 21098 ms: Timed out（译为：21098 毫秒后无法连接到 github.com 端口 443：超时）。`

````sh
/* 设置代理：*/
git config --global https.proxy
/* 取消代理：*/
git config --global --unset http.proxy
git config --global --unset https.proxy
````

`OpenSSL SSL_read:     Connection was reset, errno 10054 ...`

````sh
/* cmd 窗口输入 */
ipconfig /flushdns
````



