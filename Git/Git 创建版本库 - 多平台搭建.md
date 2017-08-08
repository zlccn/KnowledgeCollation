# Git 创建版本库 - 多平台搭建

实际工作中，你的项目不一定想让非开发人员参与，这时你就需要把你的项目私有化，虽然 <a href='https://github.com/'>GitHub</a> 是世界上最大的 Git 免费托管库，但其能让大多数人使用的都是公有库，你所上传的内容，任何人都能看见，虽然它也有私有库，却要收费，所以在你不是土豪，没有条件自己搭建托管服务器，而又想建立私有库的情况下，推荐使用 <a href='https://coding.net'>Coding</a> ,它是一个面向开发者的云端开发平台，具体好处自己问度娘，这里只说一点：**支持私有，而且免费** ，我个人觉得，所有炫酷功能和这一点比都是浮云，不是吗？

那么问题来了，如何同时在本地连接多个远程库，而又易于管理？

需要解决几个问题：

* [如何配置多个SSH Key？](#One)
* [如何将项目向多个远程库推送？](#Two)
* [如何一次 push 实现多个远程库推送？](#Three)

<br id="One"/>

## 如何配置多个 **SSH Key** ？

由于每个远程库都需要使用 Key 验证，而如果每次都覆盖了原来的 `id_rsa` ，那么之前的认证就会失效，这时就需要我们针对不同的远程库配置其专属的 SSH Key。

那么我们就面临一个重要问题，==**多用户权限**==，例：

前面我们已经连接 `github` 这个远程库，并生成密钥对(`id_rsa`、`id_rsa.pub`)，其公钥保存在了 `github` 里，每次连接时 `SSH` 客户端发送本地私钥，到服务端验证，连接服务器上保存的公钥，在这种单用户情况下，发送的私钥和公钥是配对的；

但当继续连接 `coding` 这个远程库时，就需要重新生成密钥对，这就会覆盖掉 `github` 所生成的密钥对，继而导致 `github` 的验证失败；那么我们就需要创建针对于不同远程库的独立密钥对，让其互不干扰，才能有效解决问题，

以 `github` 和 `coding` 为例：，生成新的自定义名称的公钥：

```vim
ssh-keygen -t rsa -C "you_email" -f ~/.ssh/github_isa
ssh-keygen -t rsa -C "you_email" -f ~/.ssh/coding_isa
```
执行完成后，会在 `~/.ssh/` 目录下生成 `github_isa` 、 `github_isa.pub` 、`coding_isa` 、 `coding_isa.pub` 文件；然后查看系统 `ssh-key` 代理,执行如下命令：

```
ssh-add -l
```

如果输出得到  `The agent has no identities.` 则表示没有代理。如果系统有代理，可以执行下面的命令清除代理:

```
ssh-add -D
```

然后依次将不同的ssh添加代理，执行命令如下：

```
ssh-add ~/.ssh/github_isa
ssh-add ~/.ssh/coding_isa
```

如果报错，则需要先运行一下 `ssh-agent bash` 命令后再执行 `ssh-add` 命令，完成之后，我们需要配置 `~/.ssh/config` 文件，这个文件是用于配置私钥对应的服务器，如果没有就在 `~/.ssh/` 目录下创建：

```vim
# Github
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/github_rsa


# Coding
Host git.coding.net
Hostname git-ssh.coding.net
Port 443
User git
IdentityFile ~/.ssh/coding_rsa
```

可以看到这些配置大多相同，有两点需要重点说明：

`IdentityFile` : 

配置对应的私钥地址，`~/.ssh/...` 就是路径

`Port` : 

端口修改，当你配置两个以上的远程库时，很可能出现端口被占用，这个时候就需要修改其中某一个远程库连接时所使用的端口，可以看到 `Coding` 中存在 `Port:443` ，这个端口是官方推荐的，配置成其他的也行，但建议使用官方推荐；

当全部设置完后，`:wq` 保存退出，到这里，**SSH Key** 就配置完了，可以检测配置文件是否正常工作：

```vim
ssh -T git@github.com
ssh -T git.coding.net
```

如果，正常的话，会出现如下提示：

```
Hi zlccn! You've successfully authenticated, but github does not provide shell access.
```
完成后，依照本文方法将 `coding_rsa.pub` 、 `github_rsa.pub` 公钥添加在相应的远程库里即可。


<br id="Two" />

## 如何将项目向多个远程库推送？

重温一下 `git` 仓库的设置，在指定项目中执行一下操作 ：

```
git init

echo "# learngit" >> README.md
git add README.md
git commit -m "first commit"

git remote add origin git@github.com:zlccn/learngit.git   
# 如果只想连接 Github 远程库

git remote add origin git@git.coding.net:zlccn/learngit.git
# 如果只想连接 Coding 远程库

git push -u origin master
```

从上文可以了解，在一个项目中不管你是想连接 `Github` 还是 `Coding` 都没有问题，也不需要从新在配置 **SSH Key** ，但是，当你想把一个项目 `push` 到多个 `git` 平台之中时，就要换一种玩法。

==首先我们来分析一下下面的语句：==

```
git remote add origin git@github.com:zlccn/learngit.git   
# 如果只想连接 Github 远程库
```

上面的语句是说：通过 git 的 `remote` 命令，添加一个 `github` 平台上用户为 `zlccn` 的一个名称为 `learngit` 的项目，其标签为 `origin`。

这里的重点就在于 `origin`，这是你在 clone 一个托管在 Github 上的代码库时，git 为你默认创建的指向这个远程库的**标签**，这个标签就是为了在 `push` 时，对不同平台的区分，只要在 `remote` 远程库时，命名不同的标签名称，就实现了这一效果：

```
git remote add origin git@github.com:zlccn/learngit.git   
# 连接 Github 远程库，以 origin 命名

git remote add codings git@git.coding.net:zlccn/learngit.git
# 连接 Coding 远程库，以 codings 命名

git push -u origin master
# 将修改部署到 gitHub 

git push -u codings master
# 将修改部署到 conding.net 
```


<br id="Three" />

## 如何一次 push 实现多个远程库推送？

> 某些时候，当需要推送的平台较多时，我们可能觉得一次次的更换**标签**推送过于繁琐，以此寻求能够一次 `push` 推送至多个平台的解决方案。

有些朋友可能会想到，我只需要使用 `origin` 这个标签名，去 `remote` 多个平台不就行了吗？

**想法是对的，但是操作上会出现问题**

因为，当你第二次执行 `remote` 时，会把前一次的添加覆盖掉，只会剩下一个；


好吧，既然这种方法搞不定，就来一个笨方法，**手动添加**；

在初始化 `git` 之后，当前文件夹下会建立一个 `.git` 隐藏文件，这就是本地的 `git` 仓库，在这个文件夹中有一个 `config` 文件 ：

```cmd
cd .git/
ls -ah
vim config
```

进入编辑状态后，我们可以看见其中的一些基础配置：

```
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        ignorecase = true
        precomposeunicode = true
[remote "origin"]
        url = git@git.coding.net:zlccn/learngit.git
        ...
```

上面的配置不做解释，有兴趣的朋友可以自行 Google，这里只说如何实现，可以看见 `remote` 项目中有一个 `url` ，这就是我们添加的远程库访问方式，这时 `url` 只有一条，我们只需要相对应的把 `Github` 的访问方式添加进来：

```
[remote "origin"]
        url = git@git.coding.net:zlccn/learngit.git
        url = git@github.com:zlccn/learngit.git
```

修改完成后，`:wq` 保存，至此就实现了一次操作，同时推送到多个平台的功能。当你使用 `push` 操作时，git 就会根据你设置的 `url` 顺序，向平台推送代码。


