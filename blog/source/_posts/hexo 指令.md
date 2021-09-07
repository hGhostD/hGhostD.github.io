---
layout: 待续
title: hexo 指令
date: 2019-02-24 20:18:05
tags: 
- 待续
categories: 
- 待续
---
#### 记录 hexo 常用指令

##### hexo 安装

关于搭建和环境 npm 安装就不做过多说明，可以再[官网](https://hexo.io/zh-cn/)上查到相内容。
<!-- more -->
##### 创建文件

```
$ hexo new [layout] <title>
```

新建一篇文章。如果没有设置 `layout` 的话，默认使用 [_config.yml](https://hexo.io/zh-cn/docs/configuration) 中的 `default_layout` 参数代替。如果标题包含空格的话，请使用引号括起来。

##### 发布草稿
$ hexo server
启动服务器。默认情况下，访问网址为： http://localhost:4000/。

选项	描述
-p, --port	重设端口
-s, --static	只使用静态文件
-l, --log	启动日记记录，使用覆盖记录格式

##### 生产

```
$ hexo generate
```

生成静态文件。

| 选项             | 描述                   |
| ---------------- | ---------------------- |
| `-d`, `--deploy` | 文件生成后立即部署网站 |
| `-w`, `--watch`  | 监视文件变动           |

该命令可以简写为

```
$ hexo g
```

##### 部署

```
$ hexo publish [layout] <filename>
```

该命令可以简写为：

```
$ hexo d
```