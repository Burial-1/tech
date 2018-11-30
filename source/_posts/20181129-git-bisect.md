---
title: git bisect 二分法查找问题commit
date: 2018-11-29 11:47:01
tags: [git, 二分法, debug, bisect]
category: git
---
前几天刚看到一个git command: `git bisect`，万万没想到居然这么快就用上了，真是尴尬。更尴尬的是，debug了半天最后发现是自己的commit出问题😆

在软件开发的过程中，由于测试覆盖率低，没时间回归测试等等原因，经常会遇到一种情况就是，某个feature明明前几天还好好的，不知道从什么时候开始就出Bug了……然后hmm，最笨的方法就是倒回去一个一个commit的checkout出来查找啦，稍微没那么笨的方法就是跟版本来查找，然后手动人工用二人分法（我以前就一直是这么做的，还以为自己很聪明呢，数commit数到头晕😂）

## git bisect
> This command uses a binary search algorithm to find which commit in your project’s history introduced a bug. [^1]

`git bisect`做的事情其实就跟人手做的事情本质是一样的，只不过不用自己去数commit，command会自动定位到二分法区间中点的commit并checkout

举个🌰： 
假如在develop分支中有ABCDEFG这几个commit，A最新，G最旧，已A有bug但G没有bug，想要找出A-G中最早出现bug的commit，假设最终找到有bug的commit是E。

那么首先checkout develop分支，然后
```
git bisect start
git bisect bad
```
然后 checkout G commit
```
git bisect good
```
此时git会自动checkout D commit 并显示还剩下3个commit需要检查，然后我们不停的重复以上标记`good`和`bad`的步骤，直到找出有问题的commit E，此时会显示还剩下0个commit需要检查， 记下这个commit的hash（不知道这一步有没有什么自动记下的方法？），然后
```
git bisect reset
```
之后会自动回到develop分支，然后就可以开始fixup了。

以上。

[^1]: https://git-scm.com/docs/git-bisect