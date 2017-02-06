---
layout: post
title: Pip Douban Mirror
---
The official pip repository's download speed is extemely slow in China. We could use proxy or shadowsocks,  but thanks to Douban, we have another choice.

```shell
cat ~/.pip/pip.conf
[global]
index-url = http://pypi.douban.com/simple
trusted-host = pypi.douban.com
```

enjoy~



