---
layout: post
title: "git常见用法"
date: 2015-08-21 13:52:39 +0800
comments: true
published: true
sharing: true
footer: true
categories: scm, git
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

### 3. diff

git diff

查看当前未暂存的文件与已暂存的文件中不同的部分。

git diff --cached|--staged

查看已暂存的文件与已提交的文件中不同的部分。

### 4. rm

git rm --cached

将已暂存的文件从从暂存区中移除，但不删除文件

git rm

将文件从git 暂存区中移除，并且不再跟踪此文件，最后还需要提交此次删除操作。

### 5. mv

git mv 移动文件&文件改名

git mv name1 name2 相当于执行了三条指令

```
mv name1 name2
git rm name1
git add name2
```

### 6. log

git log 查看提交记录

git log --graph 按照图形的方式显示日志记录

git log --oneline 按照单行的简化方案显示日志记录

git log -p  展开显示每次提交的内容变更

git log -p --word-diff 以单词为目标观察内容变更

git log --stat  显示增改行数统计

### 7. commit

git commit --amend 修改并更新最后一次提交内容

> --amend 参数相当于给你修改最后一次提交的机会

### 8. reset

git reset head <file>   取消暂存到 head 版本

### 9. diff

git diff

git diff target_branch_name

### 10. bundle

#### 打包

git bundle create bundle_name HEAD branch_name

#### 解包

git clone bundle_name folder_name

#### Config

  https://jk.gs/git-config.html
