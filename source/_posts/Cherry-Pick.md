---
title: Cherry-Pick的使用
description: Cherry-Pick
author: Starry
tags:
  - Tool
categories:
  - Tools
  - Git
date: 2020-09-28 20:40:55
index_img: https://img.freepik.com/free-vector/beautiful-gradient-spring-landscape_23-2148448598.jpg?w=2000
banner_img: https://img.freepik.com/free-vector/beautiful-gradient-spring-landscape_23-2148448598.jpg?w=2000
---

# Cherry-Pick的作用

假设当前所在分支为B，可以在Version Control 的 Log 中选择在A分支单个commit或者多个commit的内容，会将选中的内容拉到B分支重新进行commit，之后记得push上去，就完成了公共功能代码的拉取。

# Git Cherry-Pick的使用

1. 单个commit合并

```bash
git cherry-pick commit_id
```

2. 多个**连续**commit合并

```bash
git cherry-pick commit_id..commit_idn
```

commit_id到commit_idn之间，非闭包

```bash
git cherry-pick （commit_id..commit_idn]
```

挑选多个commit:

```bash
git cherry-pick commit_id commit_idx commit_idy
```

3. 合并过程中依次**解决冲突后**，继续合并

```bash
git cherry-pick --continue
```

4. 使用命令放弃 git cherry-pick 变更

```bash
git cherry-pick --abort 
```

# IDEA Cherry-Pick的使用

此时我有两个分支 分别为 test1分支和master分支,现要将test1分支的代码合并到master分支上.

1. 首先我们要将我们自己test分支的代码提交到库中,然后切换到master分支.

![img](https://img-blog.csdnimg.cn/630ad525782d40db837ff3b9d83d8b49.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54KO5Y2H,size_20,color_FFFFFF,t_70,g_se,x_16)

2. 通过showHistory 查看版本信息，在showHistory中的branch中查看test1分支的代码 

![img](https://img-blog.csdnimg.cn/a1393cb0f9c248e289bbb6a3d3ac3118.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54KO5Y2H,size_20,color_FFFFFF,t_70,g_se,x_16)

3. 代码冲突解决

cherry pick 后，如果存在冲突，会出现冲突提示：

界面说明：

> Accept Yours : 以自己本地代码为准。
>
> Accept Theirs : 以分支来源代码（即：test1 分支）为准。 
>
> Merge : 查看冲突文件内容，进行冲突解决。

![img](https://img-blog.csdnimg.cn/7be824cd79be4873839921a7c3cc8e67.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54KO5Y2H,size_20,color_FFFFFF,t_70,g_se,x_16)

4. 解决冲突：

冲突文件 解决界面说明：

> 最左边 ： 为你本次提交的代码。
> 中间   ： 为base 你之前拉下来的代码
> 最右边 ： 为当前服务器有改动的代码。

![img](https://img-blog.csdnimg.cn/0ecf2b5f99a947589cd31625427096c0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54KO5Y2H,size_20,color_FFFFFF,t_70,g_se,x_16)

可以选择 Accept Left(只保存提交我的改动,可能会覆盖别人的代码)。

可以点击 ">>>" 将变动的代码合并到中间的,可以保存双方的代码，或自己手动更改。

点击 Accept Right 只保存服务器上的代码 放弃自己对该类操作的提交。

冲突解决完成后，点击 apply ,则该文件已处理完成。
![img](https://img-blog.csdnimg.cn/936001d5285046b989f6e61e4a42289f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54KO5Y2H,size_20,color_FFFFFF,t_70,g_se,x_16)
