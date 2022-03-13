# 本地分支在做完修改并拉取dev分支后，push时报错的解决办法

1. **问题描述**
   
   它提示你不符合提交的规则是因为在你拉取dev分支以后系统它自动做了一个合并的commit，就是这个合并不满足条件导致的
   
   ![](/Users/jared/Library/Application%20Support/marktext/images/2021-12-07-16-11-18-image.png)

2. **解决办法**
   
   当你在拉取dev分支的时候加上`--edit参数`，即
   
   `git pull --edit origin dev`
   
   此时会弹出一个编辑页面，在顶行加上你合并的原因即可正常push。如下图：
   
   ![](/Users/jared/Library/Application%20Support/marktext/images/2021-12-08-10-31-55-image.png)
   
   （注意：这种情况适用于你刚刚拉取dev分支后push时报错，你可以通过`git log`命令回到你执行git pull origin dev命令之前，再使用`git reset --hard commit_id`重新执行`git pull --edit origin dev`命令，完整命令如下：
   
   ```git
   git log
   git reset --hard 最后一次commit_id
   git pull --edit origin dev   （编辑合并的注释信息）
   git push origin 你的分支名
   ```
   
   ）
   
   如果是你在开发过程中用`git pull origin dev`命令拉取dev分支，然后继续开发，在push时报错，很遗憾，没找到解决办法，只能回滚（你可以自己找找，说不定就找到了）！

# 记住：

**最好是在开发项目之前就拉取最新dev再新建自己的分支，或者在新建自己分支后，在拉取dev分支时主动加上--edit参数**

# git merge合并别人的分支的以后无法push的情况

解决办法：加上`--edit`参数

`git merge --edit 被合并的分支`
