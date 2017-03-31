---
title: hexo安装配置
date: 2014-1-8 22:40:58
tags: hexo
categories: hexo
---
## 安装
```
npm install -g hexo-cli
hexo init  # hexo会在目标文件夹建立网站所需要的所有文件
npm --registry=https://registry.npm.taobao.org install
hexo g # 等同于hexo generate，生成静态文件到public文件夹
hexo s # 等同于hexo server，在本地服务器运行
hexo clean #作用为清除静态文件夹的内容并删掉，主要用于更改变更了某些地方导致页面显示不完善
hexo new "title"  # 生成新文章：\source\_posts\title.md

d:\github\hexo>hexo -v
hexo: 3.2.2
hexo-cli: 1.0.2
```
### 主题采用next主题
采用版本为v5.1.0
下载地址：
https://github.com/iissnan/hexo-theme-next/releases
把下来的文件夹解压和更名为next，并复制到theme目录下

## 配置
### 压缩 CSS 文件
设置stylus属性中的compress值为true，会自动压缩 CSS 文件，hexo默认配置中不包含这一项，建议开启。如下。
```xml
theme: jacman
stylus:
  compress: true

```
### hexo中html页面
使用_config.yml下的skip_render参数。
```
skip_render:
    - `test1/*.html`
    - `test2/**`

```
### 其他配置参考
[2017年最新基于hexo搭建个人免费博客——从零开始](http://www.cduyzh.com/hexo-settings-1/)
