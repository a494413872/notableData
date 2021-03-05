---
tags: [tools]
title: sourceTree 配置多个git账号
created: '2021-02-06T10:41:20.060Z'
modified: '2021-02-06T10:46:40.853Z'
---

# sourceTree 配置多个git账号
1. 首先就不需要配置多个账号，只是需要生成一个ssh的key然后把这个ssh的公钥配置到不同的git服务器上。
2. 然后需要注意的是在sourceTree中，仓库路径要采用openssh的，路径类似于git@github.com:a494413872/hello-world.git。而不可以是https。
3. 在工具->选项->ssh 秘钥里面配置上私钥就好。

