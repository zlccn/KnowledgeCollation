# Git 常用命令 - 基本命令

## 添加

把文件添加到暂存区，* 表示把所有修改都添加到暂存区

```
git add <file> | *
```

## 查看

查看当前所有修改:

```
git status
```

如果 `git status`结果告诉你有文件被修改过，用  `git diff` 可以查看修改内容查看修改内容，输入后会打印出所有修改过的内容:

```
git diff
```


## 提交

当你使用过 `git add` 后，需要把文件提交到仓库，使用：

```
git commit -m '说明文字'
```

## 打印提交列表

按照从前到后的顺序，查看提交版本。直接使用会打印出最完整的信息以及版本号:

```
git log
```

如果只想**查找短号**和说明内容，使用命令：

```vim
git log --pretty=oneline --abbrev-commit
```

## 版本回退

`HEAD` 指向的版本就是当前版本，因此，`git` 允许我们在版本的历史之间穿梭，

想要回退到上一版本，执行命令：

```
git reset HEAD^
```

想要回退多个版本，执行命令：

```
git reset HEAD~30
```

想要回退到任意一个版本位置，执行命令：

```
git reset --hard 版本号
```

如果**只想要回退版本，但想保留修改**，使用如下命令，其修改就会保留在暂存区：

```
git reset --soft 版本号
```

如果已经 `add` 后想要撤销，则执行命令：

```
git reset
```

如果想要直接放弃修改，执行如下命令，就会清空工作区，**注意：此操作易出错，一旦误操作可查看** [Git 常见问题 - 恢复 reset --hard 删除的文件](Git%20常见问题%20-%20恢复%20reset%20--hard%20删除的文件.md)：

```
git reset --hard
```

## 打印历史提交版本

查看命令历史，以便查找制定版本的版本号，`git` 的特点是只记录修改，所以在一般情况下，你所有的历史版本都将被保存，即使你回退过的也不例外，除非你执行过删除命令

```
git reflog
```


## 合并

在多人协作的情况下，如果想将变更推送至服务器，需要先对服务器版本和本地的差异进行合并，处理冲突之后，在将本地变更推送至服务端，一般情况下，我们使用如下命令进行合并：

```
git pull --rebase origin master
```

## 推送

在确定推送的分支，其服务端与本地历史版本一致的情况下，如果想将变更推送至服务端，可以执行：

```
git push origin master
```

如果远程库维护成员只有你自己，而你又想处理服务器与本地版本的差异，想要将变更强行推送至服务器，可以执行：

```
git push --force origin master
```


## 中文文件名乱码

Git 默认中文文件名是 `\xxx\xxx` 等八进制形式，是因为对 `0x80` 以上的字符进行 `quote` ，只需要执行：

```
git config --global core.quotepath false
```   

将 `core.quotepath` 设为 false 的话，就不会对 `0x80` 以上的字符进行 quote。中文显示就正常了


## 忽略文件功能

> 实际工作中，总有一些文件是我们不希望参与提交，想要屏蔽的，而 git 恰好支持这一功能

在工作目录下创建 `.gitignore` 文件，写入忽略文件名称，最后上传版本库即可，例：

```vim
# Windows: 
Thumbs.db 
ehthumbs.db 
Desktop.ini 

# Python: 
*.py[cod] 
*.so 
*.egg 

# My configurations: 
db.ini 
deploy_key_rsa
```

如果你确实想添加该文件，可以用 `-f` 强制添加到Git：

```
git add -f <file>
```

或者你发现，可能是 `.gitignore` 写得有问题，需要找出来到底哪个规则写错了，可以用如下命令检查：

```
git check-ignore
```
















