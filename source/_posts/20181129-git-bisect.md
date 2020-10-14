---
title: git bisect 二分法查找问题commit
date: 2018-11-29 11:47:01
tags: [git, 二分法, debug, bisect]
category: git
---

前几天刚看到一个 git command: `git bisect`，万万没想到居然这么快就用上了，真是尴尬。更尴尬的是，debug 了半天最后发现是自己的 commit 出问题 😆

在软件开发的过程中，由于测试覆盖率低，没时间回归测试等等原因，经常会遇到一种情况就是，某个 feature 明明前几天还好好的，不知道从什么时候开始就出 Bug 了……然后 hmm，最笨的方法就是倒回去一个一个 commit 的 checkout 出来查找啦，稍微没那么笨的方法就是跟版本来查找，然后手动人工用二分法（我以前就一直是这么做的，还以为自己很聪明呢，数 commit 数到头晕 😂）

## git bisect

> This command uses a binary search algorithm to find which commit in your project’s history introduced a bug. [^1]

`git bisect`做的事情其实就跟人手做的事情本质是一样的，只不过不用自己去数 commit，command 会自动定位到二分法区间中点的 commit 并 checkout

举个 🌰：
假如在 develop 分支中有 ABCDEFG 这几个 commit，A 最新，G 最旧，已 A 有 bug 但 G 没有 bug，想要找出 A-G 中最早出现 bug 的 commit，假设最终找到有 bug 的 commit 是 E。

那么首先 checkout develop 分支，然后

```
git bisect start
git bisect bad
```

然后 checkout G commit

```
git bisect good
```

此时 git 会自动 checkout D commit 并显示还剩下 3 个 commit 需要检查，然后我们不停的重复以上标记`good`和`bad`的步骤，直到找出有问题的 commit E，此时会显示还剩下 0 个 commit 需要检查， 记下这个 commit 的 hash（不知道这一步有没有什么自动记下的方法？），然后

```
git bisect reset
```

之后会自动回到 develop 分支，然后就可以开始 fixup 了。

以上。

[^1]: https://git-scm.com/docs/git-bisect
