---
title: hexo博客环境配置
categories: 工具
date: 2020-03-09
---
非教程，个人博客环境配置的记录。

### 安装node.js
从官网下即可。
### 安装hexo

```
npm install hexo-cli -g
```
### Katex配置
更新next版本之后，mathjax无法显示。以前kramed的配置方法失效，更换了pandoc也不行。遂更换了Katex。

<!-- more -->

使用hexo-renderer-markdown-it-plus，详情见[next math配置文档](https://github.com/theme-next/hexo-theme-next/blob/master/docs/MATH.md)

```
npm uninstall hexo-renderer-marked
npm install hexo-renderer-markdown-it-plus
```

将 next/_config.yml中的 katex 打开。

```
math:
  ...
  katex:
    enable: true
```

执行 hexo g，报告需要安装hexo-util，执行
```
npm install hexo-util --save
```
需要注意到markdown-it有一些BUG，特别是
- 开头&后，结束&前，不允许有空格
- 开头&&前，结束&&后，不允许空格以外的字符

