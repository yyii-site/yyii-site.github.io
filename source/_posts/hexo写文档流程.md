---
title: hexo写文档流程
date: 2025-02-22 17:08:06
tags: hexo
---

远程master分支存储hexo处理后的文件，用于让github-pages显示网页。

分支hexo用来存储原始文档



## Hexo

新建文档：

```bash
hexo new xxx
```

本地服务器预览：

```bash
hexo s
```

如果 hexo 安装在远程服务器需使用 ip:4000 访问

部署到github

```bash
hexo d
```

清除本地缓存：

```bash
hexo clean
```



## 推送源文档

```bash
git add .
git commit -m "xxx"
git push origin hexo
```

