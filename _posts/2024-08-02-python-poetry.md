---
layout: post
title: "Python 依赖管理工具：Poetry"
subtitle: ""
date: 2024-08-02
mathjax:  true
author: "Yi"
header-img: "img/post-bg-2015.jpg"
tags: []
published: false
---

![alt text](assets/2024-08-02-python-poetry/image.png)

镜像：
`toml`文件中设置：

```toml
[[tool.poetry.source]]
name = "mirrors"
url = "https://pypi.tuna.tsinghua.edu.cn/simple/"
priority = "primary"
```

## Poetry安装Pytorch
