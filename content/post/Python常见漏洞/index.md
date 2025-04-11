+++
author = "Lulaide"
title = "Python常见漏洞"
date = "2025-04-11"
description = "CTF比赛中常见Python漏洞"
tags = [
    "渗透",
    "Python",
    "CTF"
]
categories = ["教程&文档"]
+++
# Python反序列化漏洞


# Python代码注入


# 网页模板渲染漏洞
> [!INFO]
> SSTI 是  Server-Side Template Injection 的缩写，意为服务端模板注入。它是指攻击者通过在输入中注入恶意代码，来操控服务器端的模板引擎，从而执行任意代码或获取敏感信息的攻击方式。SSTI 漏洞通常出现在使用模板引擎渲染网页的应用程序中，例如 Flask、Django、Jinja2 等。

| **框架**        | **默认模板引擎**                        |
|-----------------|----------------------------------------|
| Flask           | Jinja2                                 |
| Django          | Django模板引擎（Django Template Language） |
| FastAPI         | Jinja2（可选）                          |
| Pyramid         | Chameleon（默认），支持Mako、Jinja2    |
| Tornado         | Tornado模板引擎                        |
| Bottle          | SimpleTemplate（默认），支持Jinja2、Mako |
| Web2py          | Web2py模板引擎                         |
| CherryPy        | Mako（可选）                           |
| Falcon          | 无内置模板引擎（支持集成Jinja2等）      |
| Sanic           | Jinja2（可选）                          |

