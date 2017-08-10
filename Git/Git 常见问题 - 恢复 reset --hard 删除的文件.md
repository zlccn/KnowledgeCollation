# Git 恢复 reset --hard 删除的文件

大家之所以使用 Git ，就是因为其针对修改的版本控制，所以不可避免的就会频繁的**回退和转移**，实际使用中，由于种种原因，我们需要重置修改，还 Git 一片净土，一般情况下，我们使用下面的命令达到这一目的：

```vim
git reset --hard
```

但！！...

往往此时就成了悲剧的开始，这个命令就和 `sudo rm *` 一样，是有毒的，存在勿操作，而且没有确认环节，会出现原本想保留的修改也被清空的尴尬，但好在还有一定的挽回余地。

回退 `commit` 自不必说，非常简单，就算出现了错误也非常好解决，只要执行 `git reflog`，就会打印出所有commit过的历史记录，通过相应的ID进行还原就好了：

```
git reflog
```

<br/>
-------


但是对还没有 `commit` 的修改，处理起来就比较麻烦了。

和处理 `commit` 的方式一样，只要我们能获取历史记录，拿到 ID 信息，就能还原，总体来说分为两步，首先执行命令：

```
find .git/objects -type f | xargs ls -lt | sed 5q
```
这一步的主要目的是打印出指定次数的历史记录信息，最后面的 `5q` 指的是最近的 5 次，次数越大打印的越多。

然后我们就会得到下面的信息：

```vim
-r--r--r--  1 Dreamslink  staff     116  8  8 09:46 .git/objects/1f/fbea19bfa035ce1170b4bac68b209a6ea1155c
-r--r--r--  1 Dreamslink  staff     208  8  8 09:46 .git/objects/d3/2f27a1e80b97e6711daca0cd83a1fbc3d1d223
-r--r--r--  1 Dreamslink  staff      71  8  8 09:46 .git/objects/de/0b584c961d67b44089dbf8e47315fb2f27b9bd
-r--r--r--  1 Dreamslink  staff    1824  8  8 09:46 .git/objects/08/3d5ecbdaefc7d6d73ec511b43f89e85dacbaa9
-r--r--r--  1 Dreamslink  staff     210  8  8 09:45 .git/objects/59/6c4fe28a7860667b7ba37bfde94740f1e5ca60
```

可以看到，上面的信息依次按照时间重新到旧显示的非常详细，而我们所需要的 ID ，是 `.git/objects/` 之后的串，拿第一条记录在举例（直接截取示范）：

```
...   1f/fbea19bfa035ce1170b4bac68b209a6ea1155c
# 从 1f 开始，整串都是 ID ，我们在使用时需要把 1f 后的 “/” 去除
# 得到 ID： 1ffbea19bfa035ce1170b4bac68b209a6ea1155c
```
成功获取之后，我们就要根据 ID 还原文件了，与 `commit` 不同的是，我们无法直接将所有文件一次还原出来，需要将每个 ID 所示的单个文件读取出来，重定向保存到自定义的文档中去，使用命令:

```
git cat-file -p ID > url/file.md
```
上面命令中 `ID` 自不必说，主要说一下 `url/file.md` ，`url` 是路径，可以不填*（将文档直接保存在当前命令所在目录）* ，`file.md` 就是文档名称了，从 ID 中取到的文件内容即保存在你所创建的文档中，至此，我们就实现了这一功能。

> 注意：由于 git 只记录修改，所以这种方法只能还原你 `add` 过的文件，否则是没有办法可以还原的，因为 git 并未记录你的修改（当然，也可能是本人孤陋寡闻，如果有，欢迎分享）。
> 
> 所以在使用 `git reset --hard` 这一语句的时候，要格外谨慎。
    

