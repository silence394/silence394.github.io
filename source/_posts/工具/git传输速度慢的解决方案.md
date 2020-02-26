---
title: git传输速度慢的解决方案
categories: 工具
---

在用git的时候，经常会碰到push、pull、clone速度很慢的情况，并且当传输量过大的时候很有可能失败。这是因为git的域名被限制了。 

通过以下步骤可以提升git传输的速率。

1. 通过[查询域名IP](https://www.ip.cn/index.php)查询github.com和global-ssl.fastly.Net的IP地址。
2. 将查的的IP添加到hosts的末尾。
```
151.101.73.194 github.global.ssl.fastly.net
52.74.223.119 github.com
```
3. 执行ipconfig /flushdns，刷新DNS。
4. 重新启动git。

对于使用s的可以配置git如下:：
``` sh
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
```