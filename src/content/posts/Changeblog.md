---
title: Change Blog
published: 2026-03-30
description: 一个切换博客新内容
encrypted: true
# pinned: true
passwordHint: "宇宙的真理是什么？"
password: "42"
# alias: "encrypted-example"

tags: ["first", "Encryption", "宇宙の起源"]
category: "blog"
draft: false
---

## 使用教程

:::important
启动本地项目
:::
```bash
pnpm dev
```
完成之后在浏览器访问 [http://localhost:4321](http://localhost:4321) 来查看 `你的博客`

:::important
运行下面的这个代码将网站打包成静态文件，会生成并保存在 **dist** 目录里
:::

```bash
pnpm build
```
生成的 **dist** 可以部署到服务器上，也可以放置在 **github** 这个项目是没有后端的，无需数据库，所以需要手动配置前端文件
