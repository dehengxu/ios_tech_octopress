---
layout: post
title: "git常见用法"
date: 2015-08-21 13:52:39 +0800
comments: true
published: true
sharing: true
footer: true
categories: 
---

### 1.checkout

#### 切换到指定 commit 

```
checkout sha-1
```

#### 切换到分支

```
checkout branch_name
```

---

### 2.merge

```
   D--------E
  /
 /
A---B---C---F----G master
```

```
git merge test
```

合并结果：

```
   D-------------E
  /               \
 /                 \
A---B---C---F----G test, master
```

---

### 3.rebase

```
   D--------E
  /
 /
A---B---C---F----G master
```


```
git rebase dev
```

```
A---B---D---C---E---F----G  test, master
```

复位基线，将两个分支的提交纪录，合并到一条基线路线上。功能作用上和 merge 类似，但是不会产生新的提交节点，而且不存在多个提交路线。

当出现冲突的时候，先去解决冲突，完成后，使用 `rebase --continue` 继续 `rebase` 操作。如果冲突不重要也可以 `rebase --skip` 忽略冲突。






