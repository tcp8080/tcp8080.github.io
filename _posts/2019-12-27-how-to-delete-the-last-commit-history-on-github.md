---
layout: post
title:  "How to delete the last commit on github"
date:   2019-12-27 22:19:43 +0800
categories: tip
---


#### 问题：有时候往github提交了代码之后，发现由于疏忽把个人隐私上传了，需要删除这个提交。
##### Question: Some private infomation was pushed into github by mistake, and we want to delete this commit.

#### 用以下命令行：
##### Here is what we do in cmd:

```shell
git rebase -i HEAD~2
git push -f origin HEAD^:master
```