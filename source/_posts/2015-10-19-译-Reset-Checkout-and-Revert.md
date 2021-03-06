title: '[译]Reset, Checkout, and Revert'
date: 2015-10-19 23:11:30
tags: Git
categories: Git
---
翻译自[这篇文章](https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting/file-level-operations)

`git reset`, `get checkout` 和 `git revert`命令是你Git工具箱中最有用的几个工具。它们可以让你在仓库中撤销某些改变，前两个命令也可以用来处理提交或者单个的文件。

因为他们很相似，所以在特定的开发场景下，使用哪种命令就很容易混淆。在这篇文章中，我们将比较这三个命令常见的配置，希望你能自行的用这些命令去驾驭你的仓库。

{%asset_img 01.svg%}

这帮助我们去想每一个命令在这三个Git仓库主要组件（工作区，快照，提交历史）中的作用。当你读这篇文章的时候，时常回想这些组件。

# 提交层面的操作

你传给 `git reset` 和 `git checkout` 的参数决定了它们的作用域。当你不包含一个文件路径做参数时，它们作用于整个提交。这是我们在这一章节将要介绍的内容。注意 `git revert`并没有文件层面对应的内容。

## Reset

在提交层面，重设是移动分支末端到另一个不同的提交的一种方式。这能用来从当前分支移走一些提交。比如，下面的命令把 hotfix 分支向后移动两个提交。

```
git checkout hotfix
git reset HEAD~2
```

这两个位于hotfix末端的提交现在正在摇摆，也就是说，它们将会在Git下次执行垃圾收集的时候删除。换言之，你可以说你想要抛弃这些提交。下图看起来会比较直观：

{%asset_img 02.svg%}

`git reset` 的使用是一个简单的方式去撤销还未分享给他人的更改。这是你将要用到的命令当你已经开始在一个feature分支上工作，然后你发现自己一直在想，“卧槽，我在干啥？我应该从头开始。”

另外，移动当前分支时，你也可以通过传递一下的某个标识，让 `git reset` 去修改快照区和／或工作区。

1, --soft － 快照区和工作区不会被修改。
2, --mixed －快照区将会更新到特定的提交，但是工作区不会受影响。这是默认选项。
3, --hard －快照区和工作区都会更新到特定的提交。

当定义这个 `git reset` 操作的作用域时，它将会容易去思考这些模式。

{%asset_img 03.svg%}

这些标识常常和 `HEAD` 参数一起使用。比如 `git reset --mixed HEAD` 清空快照区，但是保留工作区的更改。另一方面，如果你想完全抛弃所有未提交的更改，你可以使用 `git reset --hard HEAD`. 这是 `git reset` 最常用两种使用方式。


当传入的提交是 `HEAD` 以外的参数时，一定到小心，因为这将会重写当前分支的历史。正如在[The Golden Rule of Rebasing](我并不知道地址是啥)中提到的，当工作在一个公共分支时，这将是一个大问题。

## Checkout

到现在为止，你应该比较熟悉 `git checkout` 的提交层面。当传递一个分支名称时，它让你在不同分支间切换。

```
git checkout hotfix
```

本质上讲，这个命令所做的就是移动HEAD到另一个分支，并且更新工作区使之于其匹配。因为这有重写本地更改的风险，Git强制你提交或者暂存工作区里的任何更改，以防这些更改在checkout操作中丢失。并不像 `git reset` 那样，`git checkout` 并不会向前后移动任何分支。

{%asset_img 04.svg%}

你也可以检出任何提交通过传递一个提交引用而不是分支。这确实也像检出一个分支那样：它移动这个HEAD引用到特定的提交。比如，下面的命令将检出当前提交的父父层。

```
git checkout HEAD~2
```

{%asset_img 05.svg%}

这对于快速查看项目中老的版本十分有用。但是，由于对于当前的HEAD并没有指定分支引用，这将把你放进一个临时的HEAD状态。如果此时你增加新的提交，那就会比较危险，因为当你切换到另一个分支之后，你就没有办法再找到它们。因为这个原因，在你增加新的提交到一个暂存的HEAD状态前，你应该新建一个新分支。

## Revert

恢复原状通过提交一个新的提交去撤销一个提交。这是撤销改变的一个安全的方式，因为它不会改变或者重写提交的历史。比如，下面的命令将会找出最后两个提交的改变，创建新的提交去撤销那些改变，并且把这个新的提交钉在这个项目上。

```
git checkout hotfix
git revert HEAD~2
```

下图更直观：

{%asset_img 06.svg%}

和 `git reset` 做比较，这个命令并不修改已存在的提交历史。因为这个原因，`git revert` 应该在公共分支上使用来撤销更改，`git reset` 应该在个人分支上使用来撤销更改。

你也可以认为 `git revert` 是撤销已提交更改的工具，而 `git reset HEAD` 是撤销未提交更改的工具。

像 `git checkout`，`git revert` 都有重写工作区文件的风险，所以当你进行这些操作时，都会被要求提交或缓存更改。

# 文件层面的操作

这个 `git reset` 和 `git checkout` 命令也允许一个可选的文件路径作为参数。这将修改它们的行为。代替操作整个快照区，这种方式将会限制它们操作在单个文件中。

## Reset

当传递一个文件路径调用时， `git reset` 更新这个快照区去匹配特定提交的版本。比如，下面命令将会取得foo.py的倒数第二次提交的版本，并且在下一次提交时存储它。

```
git reset HEAD~2 foo.py
```

和提交层面的 `git reset` 一样，和HEAD搭配使用是比较常见的。`git reset HEAD foo.py` 将会取消暂存foo.py. 但是它所包含的更改在工作区中还是可见的。

{%asset_img 07.svg%}

这个 `--soft`, `--mixed` 和 `--hard` 标识对文件层面的 `git reset` 并没有什么卵用。因为快照区一直在更新，而工作区从来不更新。

## Checkout
检出一个文件和使用`git reset`后面带一个文件路径是类似的，唯一的不同是，它更新的是工作区而不是暂存区。并不像提交等级的版本那样，这个命令并不会移动`HEAD`的引用，这意味着你将不会切换分支。

{%asset_img 08.svg%}

比如，下面的命令将会使在工作区里的foo.py文件匹配倒数第二次提交的这个文件。

```
git checkout HEAD~2 foo.py
```
像提交等级的`git checkout`命令一样，这被用来检查一个项目的老的版本——但是他的作用范围是指定的文件。

如果你暂存和提交这个检出的文件，这还会有回退到老文件的作用。注意这将移除所有随后的改变。但是`git revert`命令仅仅撤销某个指定的提交的改变。

像`git reset`一样，这个命令常常和HEAD一起使用。比如，`git checkout HEAD foo.py`有丢弃未暂存的更改的作用。这有点类似`git reset HEAD --hard`,但是它操作的是指定的文件。

## 总结
你现在应该拥有所有工具去撤销改变了。`git reset`,`git checkout`,`git revert`命令可能是让人迷惑的，但是当你想一想它们对工作区，暂存区和提交历史的作用，对于那种命令适合哪种开发任务将会变的容易区别。

下面的表格总结一些常用的用例，确保把这个手册放在手边，因为在你的Git生涯中，无疑你会用到它们中的一个或多个。

Command   | Scope         | Common use cases |
----------| --------------|-------------------|
git reset |Commit-level   |Discard commits in a private branch or throw away uncommited changes|
git reset |File-level     |Unstage a file|
git checkout|Commit-level|Switch between branches or inspect old snapshots|
git checkout|File-level|Discard changes in the working directory|
git revert|Commit-level|Undo commits in a public branch|
git revert|File-level|(N/A)|





