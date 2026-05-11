---
layout: 待续
title: hexo 指令
date: 2019-02-24 20:18:05
tags: 
- 待续
categories: 
- 待续
tog_img: https://s2.loli.net/2022/01/07/PX3iRIcTzZUJBb7.jpg
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

###### git 相关
git 校验有效性
```
ssh -T git@github.com
```

git 冲突解决
```
git fetch origin
git reset --hard origin/master
```

##### Butterfly 主题
文档地址：https://butterfly.js.org/posts/21cfbf15/

安装使用的时候遇到了 `WARN  No layout: index.html` 这个提示，是由于 theme 文件夹下没有下载主题导致的。安装主题有两种方式，一种是通过 git 安装主题文件，另一种是通过 npm 安装。后来使用了 npm  的方式安装，感觉比较简洁，复制 _config.butterfly.yml 配置文件。

~~在使用的是遇到了 git 无法提交的问题，使用公司的电脑提交 butterfly 的配置文件无法保存到 Git 上。可能是由于 Git 忽略了吧，需要进行修正。~~ 
 提交主题配置文件应该在根目录下，不应该修改 npm 目录下的 _config.yml 文件。
 ![主题配置文件](https://s2.loli.net/2022/01/07/y618Kc3t4S9FzIC.png)