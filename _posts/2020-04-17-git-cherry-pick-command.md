---
layout: post
title: Git - cherry-pick 命令
date: 2020-04-17
Author: Hyperzsb
tags: [git]
comments: true
toc: true
---

cherry-pick同rebase相似，同是用现有提交对版本库进行更改。

本文将对git cherry-pick命令官方文档进行基本翻译。

<!-- more -->

---

## 语法

```shell
$ git cherry-pick [--edit] [-n] [-m parent-number] [-s] [-x] [--ff]
		  [-S[<keyid>]] <commit>…
$ git cherry-pick (--continue | --skip | --abort | --quit)
```

---

## 说明

**给定一个或多个现有的提交记录，引入这些记录所作的更改，并为每一个引入创建一个新提交。**这个操作需要你的工作树是干净的。

需要注意的是：

- 当前分支和`HEAD`指针将指向最后一个成功应用的提交；
- `CHERRY_PICK_HEAD`指针将指向所引入的更改**难以被应用**的提交，如出现冲突；
- 应用的更改将在你的索引和工作树中同时更新；
- 对于产生冲突的文件，索引将记录最多三个版本（**注意：此处说法存疑，如何获取这三个版本方法不详，请自行查询**）。关于如何处理冲突请参阅`git merge`命令文档；
- **不会有不存在于这些提交的其它的更改被引入。**

------

## 选项

### \<commit\>...

#### 原文翻译

**用于`cherry-pick`操作的提交名。**可以使用提交名的不同表示形式来指定一个或多个提交。

> Sets of commits can be passed but no traversal is done by default, as if the `--no-walk` option was specified, see git-rev-list. Note that specifying a range will feed all \<commit\>… arguments to a single revision walk.
>
> **此段说明具体含义不甚明确，有待后续翻译说明。**

#### 具体说明

***后续更新中......***

------

### -e / --edit

#### 原文翻译

用于编辑先前的提交说明。

#### 具体说明

**这是一个很易于理解的行为，略。**

------

### --cleanup=\<mode\>

#### 原文翻译

用于决定提交信息在进入提交进程前将以何种方式被清除。更多信息参考`git commit`命令文档。

> In particular, if the \<mode\> is given a value of `scissors`, scissors will be appended to `MERGE_MSG` before being passed on in the case of a conflict.
>
> **此段说明具体含义不甚明确，有待后续翻译说明。**

#### 具体说明

***后续更新中......***

------

### -x

#### 原文翻译

创建提交时将自动添加一行**“(cherry picked from commit …)”**信息到初始提交信息上，用来说明哪些提交是通过`cherry-pick`命令引入的。这一操作只会在没有冲突的提交上完成。

如果你从你的私有分支进行`cherry-pick`操作，**不要**使用这个选项，因为这对于接收方来说是无效的（**注意：此处说法存疑**）。另一方面如果你从两个公开的分支进行`cherry-pick`操作，例如将开发分支上的一个修复应用到主分支时，这将会很有用。

#### 具体说明

**这是一个很易于理解的行为，略。**

------

### -r

#### 原文翻译

在以前的版本中，效果同`-x`参数。**已经被弃用。**

#### 具体说明

**这是一个很易于理解的行为，略。**

------

### -m parent-number / --mainline parent-number

#### 原文翻译

通常情况下你不能够在一个合并提交上进行`cherry-pick`操作，因为你不能够明确合并的哪一边是工作主线。这个参数确定了工作主线的父提交号是哪一个（从`1`开始）并且允许`cherry-pick`根据选定的父提交重演变化的过程。

#### 具体说明

***后续更新中......***

------

### -n / --no-commit

#### 原文翻译

通常情况下`cherry-pick`命令将会自动地创建一系列提交。这个参数的使用将在你的索引和工作树上应用每一个命名的提交（**译注：即你所选定执行`cherry-pick`操作的提交**）所包含的必要修改，**但不创建提交**。事实上，当使用这个选项时，你的索引不必要匹配`HEAD`指针所指向的内容，在这种情况下，`cherry-pick`将不考虑你的索引的初始状态。

在你要对获取来自多个提交的更改到你的索引时，这将很有用。

#### 具体说明

***后续更新中......***

------

### -s / --signoff

#### 原文翻译

向每一个提交信息末尾添加一行`Signed-off-by`说明。

关于`signoff`选项的具体说明，请参考`git commit`命令文档。

> **有待深入说明。**

#### 具体说明

***后续更新中......***

------

### -S[\<keyid\>] / --gpg-sign[=\<keyid\>]

#### 原文翻译

使用GPG密钥对提交进行签名。`keyid`参数是可选的并且默认于提交者的身份，如果选定了这个参数，必须紧挨着选项名而不能有空格（**译注：例如`-Saf3d12dab47e`和`--gpg-sign=af3d12dab47e`**）。

#### 具体说明

***后续更新中......***

------

### -ff

#### 原文翻译

如果当前`HEAD`指针同要执行`cherry-pick`操作的提交的父提交一致，将直接执行**快进（fast forward）**操作。

#### 具体说明

***后续更新中......***

------

### --allow-empty

#### 原文翻译

默认情况下，对一个空的提交进行`cherry-pick`操作会失败，表示需要显示地调用`git commit --allow-empty`。这个选项覆写了这种行为，即允许空提交在`cherry-pick`操作中被自动保存。**值得注意的是**，如果`--ff`选项被启用，空的提交在不指定`--allow-empty`情况下也会被保留。**同样值得注意的是**，这个选项只会保留初始即为空的提交（即这个提交记录了同它父提交相同的变更树）。那些由于之前的提交存在而变为冗余的提交将被抛弃。如果想强制引入这些提交，请使用[`--keep-redundant-commits`](#keep-redundant-commits)。

#### 具体说明

***后续更新中......***

------

### --allow-empty-message

#### 原文翻译

默认情况下，对一个提交信息为空的提交进行`cherry-pick`操作会失败。这个选项覆写了这种行为，即允许提交信息为空的提交执行`cherry-pick`操作。

#### 具体说明

**这是一个很易于理解的行为，略。**

------

<a id="keep-redundant-commits"></a>

### --keep-redundant-commits

#### 原文翻译

如果一个提交重复了已经存在于历史记录中的先前的提交，那么它就会成为空的**冗余提交**。默认情况下，这些冗余提交将会使得`cherry-pick`操作停止以使得用户验证该提交。这个选项覆写了这种行为，即为该提交创建空的提交对象。该选项囊括了`--allow-empty`选项的功能。

#### 具体说明

***后续更新中......***

------

### --strategy=\<strategy\>

#### 原文翻译

使用给定的合并策略，这个选项只能被使用一次。

关于合并策略的信息，请参考`git merge`命令文档。

#### 具体说明

**这是一个很易于理解的行为，略。**

------

### -X\<option\> / --strategy-option=\<option\>

#### 原文翻译

传递特定合并策略的参数到合并策略中去。

关于合并策略及相关参数的信息，请参考`git merge`命令文档。

#### 具体说明

***后续更新中......***

------

### --rerere-autoupdate / --no-rerere-autoupdate

#### 原文翻译

> Allow the rerere mechanism to update the index with the result of auto-conflict resolution if possible.
>
> **此段说明具体含义不甚明确，有待后续翻译说明。**

#### 具体说明

***后续更新中......***

---

## 子命令

### --continue

#### 原文翻译

在解决了`cherry-pick`命令产生的冲突后，根据`.git/sequencer`文件中的信息，继续执行`cherry-pick`操作。

#### 具体说明

**这是一个很易于理解的行为，略。**

------

### --skip

#### 原文翻译

跳过当前提交（**译注：例如产生冲突的提交**），继续执行后续`cherry-pick`操作。

#### 具体说明

**这是一个很易于理解的行为，略。**

------

### --quit

#### 原文翻译

忘记当前操作进程。可以在`cherry-pick`失败后用来清理序列状态。

#### 具体说明

***后续更新中......***

------

### --abort

#### 原文翻译

取消当前`cherry-pick`操作并且回滚至初始状态。

#### 具体说明

**这是一个很易于理解的行为，略。**

---

## 样例

```shell
$ git cherry-pick master
# 应用由主分支顶端的提交引入的更改，并使用此更改创建新的提交
$ git cherry-pick ..master
$ git cherry-pick ^HEAD master
# 应用所有来自于master分支的祖先提交而不是HEAD指针的祖先提交
#（译注：即取master分支与当前分支的差集）
# 所引入的更改来生成新的提交
$ git cherry-pick maint next ^master
$ git cherry-pick maint master..next
# 应用所有提交所引入的更改，这些提交是maint或next的祖先，但不是master或其任何祖先
# 注意，后者并不意味着maint以及master和next之间的所有内容
# 特别的，如果maint包含在master中，则不会使用它
$ git cherry-pick master~4 master~2
# 应用master指向的第五次和第三次最后提交所引入的更改，并使用这些更改创建两个新提交
$ git cherry-pick -n master~1 next
# 将由master指向的第二个最后一次提交和next指向的最后一次提交引入的更改应用到工作树和索引上
# 但不使用这些更改创建任何提交
$ git cherry-pick --ff ..next
# 如果历史是线性的，并且HEAD是next的祖先，则更新工作树并将HEAD指针快进到直到匹配next
# 否则，将next中的提交（而不是HEAD）引入的更改应用于当前分支，为每个新更改创建新的提交
$ git rev-list --reverse master -- README | git cherry-pick -n --stdin
# 将主分支上的所有涉及到README文件的提交所引入的更改应用于工作树和索引
# 以便在合适的情况下对其进行检查并将其生成单个新提交
```

## 参考

Git cherry-pick 官方文档