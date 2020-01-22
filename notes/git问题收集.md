---
title: git问题收集
created: '2019-12-22T06:52:55.763Z'
modified: '2019-12-22T07:02:17.109Z'
---

# git问题收集
- 报错 fatal: refusing to merge unrelated histories
原因是，本地分支和远程分支被认为是没有关系的两个分支
解决方案git pull origin master --allow-unrelated-histories
