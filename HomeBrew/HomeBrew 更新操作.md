# Homebrew 更新操作

[Homebrew](https://brew.sh/index_zh-cn.html) 是 macOS 的软件包管理器，使用 `Homebrew` 可以安装 Apple 没有预装，但你需要的东西，通常用作**全局安装**开发类工具框架依赖包。

具体安装方法请自行访问 [官方网站](https://brew.sh/index_zh-cn.html) 查询，本文话题围绕 **时常定期更新** `Homebrew`，主要为了帮我记录常用的命令方便以后查阅，也希望它也能帮到你。

日常工作中，很多朋友都会忘记更新工具版本，只是到了用到某个工具的新特性，或者是为了修复软件的漏洞，也或许新安装的包非要依赖另一个包的新版本时，才会去考虑更新相对应的工具版本，但这时，由于很多工具都停留在许久之前的历史版本，单独去更新某一个，就有可能会出现连锁反应，导致环境出错，而这种问题如果出现，排查解决是比较坑爹的，最后只能导致拆掉所有，从新去安装；

所以为了保证当前环境足够稳定，应当定期的更新所有依赖。

<br/>

## 更新 Homebrew

要获取最新的包列表，首先要更新 `Homebrew` 自己：

```
brew update
```

<br/>

## 更新包（formula）

更新之前，首先用 `brew outdated` 查看哪些包需要更新

```
brew outdated
```

然后就可以用 `brew upgrade` 去更新，这时 `Homebrew` 会安装新版本的包，==但注意此时旧版本仍然会保留。==

```
brew upgrade            # 更新所有的包
brew upgrade $formula   # 更新指定的包
```

<br/>

## 清理旧版本

新版本安装后，旧版本在一般情况下没有留存的必要，这时用 `brew cleanup` 清理掉旧版本和缓存文件，==Homebrew 只会清除比当前版本更老的版本文件，所以不用担心有些包没更新会被删除。==

```
brew cleanup            # 清理所有包的旧版本
brew cleanup $formula   # 清理指定包的旧版本
brew cleanup -n         # 查看可清理的旧版本包，不执行实际操作
```

<br/>

## 锁定不想更新的包

果经常更新的话，`brew update` 一次更新所有的包是非常方便的。但我们有时候会担心自动升级把一些不希望更新的包更新了。数据库就属于这一类，尤其是 `PostgreSQL` 跨 `minor` 版本升级都要迁移数据库的。我们更希望找个时间单独处理它。这时可用 `brew pin` 去锁定这个包，然后 `brew update` 就会略过它了。

```
brew pin $formula       # 锁定某个包
brew unpin $formula     # 取消锁定
```

<br/>

## 其他常用命令

`brew info` 可以查看包的相关信息，最有用的应该是包依赖和相应的命令。比如 Nginx 会提醒你怎么加 `launchctl` ，`PostgreSQL` 会告诉你如何迁移数据库。这些信息会在包安装完成后自动显示，如果忘了的话可以用这个命令很方便地查看。

```
brew info $formula      # 显示某个包信息
brew info               # 显示安装包的数量，文件数量，和总占用空间
```

`brew deps` 可以显示包的依赖关系，我常用它来查看已安装的包的依赖，然后判断哪些包是可以安全删除的。

```
brew deps --installed --tree  # 查看已安装的包的依赖，树形显示
```

此外还有很多有用的命令和参数，没事 `man brew` 一下可以涨不少知识。
