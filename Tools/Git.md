# Git

## Git基本理论
- ### 三个区域
    ![image](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p0NJ4L9OPI9ia1MmibpvDd6cSddBdvrlbdEtyEOrh4CKnWVibyfCHa3lzXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

    - 工作区(Working Directory):用户平时存放项目代码的地方。 
    - 暂存区(Stage/index)：零时存放用户改动的地方，事实上他只是个文件，保存即将提交的文件列表信息
    - 仓库区/本地仓库(Repository)：安全存放数据的位置这里存放用户提交所有版本的数据
    - 远程仓库(Remote)：托管代码的服务器
- ### 工作流程
    ![image](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p09iaOhl0dACfLrMwNbDzucGQ30s3HnsiaczfcR6dC9OehicuwibKuHjRlzg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
    - 在工作目录中添加、修改文件;
    - 将需要进行版本管理的文件放入暂存区域;
    - 将暂存区域的文件提交到Git仓库
- ### 文件的四种状态
    - **Untracked**: 未跟踪, 此文件在文件夹中, 但并没有加入到git库, 不参与版本控制. 通过==git add== 状态变为==Staged==.
    - **Unmodify**: 文件已经入库, 未修改, 即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改, 而变为==Modified==. 如果使用==git rm==移出版本库, 则成为Untracked文件
    - **Modified**: 文件已修改, 仅仅是修改, 并没有进行其他的操作. 这个文件也有两个去处, 通过==git add==可进入暂存==staged==状态, 使用==git checkout==则丢弃修改过, 返回到==unmodify==状态, 这个==git checkout==即从库中取出文件, 覆盖当前修改 !
    - **Staged**: 暂存状态. 执行==git commit==则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为==Unmodify==状态. 执行==git reset HEAD filename==取消暂存, 文件状态为==Modified==

    ```
    #查看指定文件状态
    git status [filename]
    
    #查看所有文件状态
    git status
    
    #添加所有文件到暂存区
    git add .   
    #提交暂存区中的内容到本地仓库 -m 提交信息
    git commit -m "消息内容"    
    ```
- ### 忽略文件
    当我们不想把某些文件纳入到版本控制中的时候,在主目录下建立".gitignore"文件,此文件有如下规则
    - 忽略文件中的空行或以井号（#）开始的行将会被忽略。
    - 可以使用Linux通配符。例如：星号（*）代表任意多个字符，问号（？）代表一个字符，方括号（[abc]）代表可选字符范围，大括号（{string1,string2,...}）代表可选的字符串等。
    - 如果名称的最前面有一个感叹号（!），表示例外规则，将不被忽略。
    - 如果名称的最前面是一个路径分隔符（/），表示要忽略的文件在此目录下，而子目录中的文件不忽略。
    - 如果名称的最后面是一个路径分隔符（/），表示要忽略的是此目录下该名称的子目录，而非文件（默认文件或目录都忽略）。
    ```
    #为注释
    *.txt        #忽略所有 .txt结尾的文件,这样的话上传就不会被选中！
    !lib.txt     #但lib.txt除外
    /temp        #仅忽略项目根目录下的TODO文件,不包括其它目录temp
    build/       #忽略build/目录下的所有文件
    doc/*.txt    #会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
    ```

## 基本操作

- 安装

  ```
  yum install git
  ```

- 设置用户名以及联系邮箱

  ```
  git config --global user.name xxx           #设置用户名xxx 
  git config --global user.email xxx@xxx.com  #设置邮箱
  ```

- init 初始化本地仓库

  ```
  # 直接指定目录进行创建目录 
  git init /home/git/test 
  # cd到指定的目录,直接
  init cd /home/git/test 
  git init 
  ```

  

- 删除本地仓库

  ```
  # 直接删除当前目录下的.git 文件即可 
  rm -rf .git
  ```

  

- status 查看当前仓库的状态

  ```
  git status
  ```

- add 添加 代码/文件 到 暂存区

  ```
  git add 1.txt 2.txt 
  git add .           #添加当前整个目录
  ```

- commit 把代码保存提交到仓库

  ```
  git commit readme.md -m "这里是提交的备注"
  ```

- rm 删除文件

  ```
  git rm -f test.txt
  ```

- mv 移动或改名

  ```
  git mv 1.txt demo/2.txt  #移动且改名
  ```

- log 日志

  ```
  git log                  #查看项目的日志 
  git log test.txt         #查看文件的日志 
  git log --pretty=oneline #让日志单行显示
  ```

- reflog 版本控制

  ```
  git reflog            #查看版本变化 
  git reset --hard HEAD^^  #往后推^个版本 
  git reset --hard abb32   #根据版本号切换版本
  ```

- 忽略一些 文件/文件夹 不提交

  ```
  创建一个 .gitignore 的文件, 里面写入要忽略的 文件/文件夹 名 即可.
  ```

### 分支管理 branch

- **查看分支**

git branch #其中 master 为主分支 

![img](D:\Study\note\qqEC3EE7D5AA0FB3737AA53A762F71302A\dc2cb48f00c043f5981e5bff408bb505\clipboard.png)

**表示当前分支为 master** 

- **创建分支**

```
#git branch <name> 
git branch demo    
```

+ **checkout 切换分支**

```
git checkout master
```

- **merge 合并分支**

```
git checkout master #一般先切换到主分支 
git merge demo      #把demo分支合并到主分支 
#如果创建分支后 主分支有改动,合并分支是会有冲突,需要手动解决 
```

- **删除分支**

```
git branch -d demo
```

- **快速创建分支且切换到新的分支**

```
git checkout -b demo
```

### 远程仓库 remote

- **查看远程仓库**

```
git remote git remote -v    #查看远程仓库地址
```

- **添加远程仓库链接地址**

```
# git remote <name> <url> 
git remote add mywork https://xxxxxx.com/xxx.git  #把远程仓库的地址起个别名为 mywork ,并记录下来
```

一般来说，如果当前目录不是Git仓库，这条命令是无法执行的，既然是远程管理，那么这条命令有点鸡肋。

- **修改远程库的名称**

```
git remote rename <old name> <new name> 
```

- **clone 复制 远程仓库 到本地**

```
#把文件下到当前目录,会生成一个test的文件夹 
git clone https://github.com/Ameng-cs/MyNotes.git
#或者 
git clone demo #demo 是已添加的远程库地址
```

拿到远程仓库的地址后第一件事就是克隆到本地，然后才用 推push、拉pull，如果是直接用连接生成仓库后会自动有一个默认的地址别名，origin

- **push 把代码推到远程库**

```
# git push <远程库地址> <分支名> git push demo master 
git push https://www.gitee.com/demo.git master 
#也可以直接push,它会推向当前默认的的地址和分支 
git push
```

在推向远程仓库之前要在本地commit

- **fetch 获取最新版本到 本地新的分支**

```
git fetch origin master:tmp #从远程仓库master分支获取最新，在本地建立tmp分支 
git diff tmp                #将当前分支和tmp进行对比
git merge tmp               #合并tmp分支到当前分支
```

- **pull 更新本地仓库并合并分支**

```
#git pull命令的作用是：取回远程主机某个分支的更新，再与本地的指定分支合并。 
git pull <远程库地址> <分支名>:<本地分支名> 
#本地分支名可以省略,默认合并到当前分支 
git pull demo master
```

一句话总结 git pull 和 git fetch 的区别：**git pull = git fetch + git merge**

**git fetch不会进行合并执行后需要手动执行git merge合并分支，而git pull拉取远程分之后直接与本地分支进行合并。更准确地说，git pull使用给定的参数运行git fetch，并调用git merge将检索到的分支头合并到当前分支中。**

**上面的pull操作用fetch表示为：**

```
git fetch origin master:brantest 
git merge brantest
```

**相比起来`git fetch`更安全一些，因为在merge前，我们可以查看更新情况，然后再决定是否合并。**

- **配置公钥免密登录远程仓库**

  - **配置ssh格式的远程仓库地址**

    ```
    git remote add test git@git.oschina.net:lianshou/test.git
    ```

  - **创建ssh key**

    ```
    ssh-keygen -t rsa -C "youremail@example.com" # 把邮件地址换成你自己的邮件地址,一直回车,不用输入密码. 
    # 完成后,可以在用户主目录里找到.ssh目录,内有id_rsa和id_rsa.pub两个文件. 
    # id_rsa是私钥,id_rsa.pub是公钥.
    # 这两把钥匙是成对的,可以让分别持有私钥和公钥的双方相互认识.
    ```

  - **把公钥放到服务器**

用记事本打开id_rsa.pub,复制公钥内容. 登陆git.oschina.net,如下图,填入公钥并保存.

 

 

![img](D:\Study\note\qqEC3EE7D5AA0FB3737AA53A762F71302A\4b863a1363f74ce787cfcb00d4d02686\clipboard.png)