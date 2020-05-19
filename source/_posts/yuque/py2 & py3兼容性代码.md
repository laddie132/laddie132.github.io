---
title: py2 &amp; py3兼容性代码
urlname: py2-py3-compatibility
date: 2020-05-19 08:50:51 +0800
categories: [Dev]
tags: [Python]
---

```python
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
from __future__ import unicode_literals

import sys
if sys.version_info[0] < 3:
    reload(sys)
    sys.setdefaultencoding('utf-8')
```

<!-- more -->

## 编码问题

`from __future__ import unicode_literals`

- python 中存在两种编码 bytes 和 unicode，分别使用 b''和 u''显示指定
  - 使用 bytes 编码，在计算字符串长度时，看的是字节长度，而不是实际长度；而 unicode 没有这个问题
  - encode 操作将 unicode 转为 bytes；decode 操作将 bytes 转为 unicode，从而实现不同编码格式之间的转换
- py2 默认字符串 str 使用 bytes 编码，py3 默认全部使用 unicode 编码
- py2 默认使用 ascii 编码格式，py3 默认使用 utf-8 格式

## 绝对引入

`from __future__ import absolute_import`

- py2 引入包时，直接从当前目录引入；py3 默认从环境变量引入

## 除法

`from __future__ import division`

- py2 默认除法为整除；py3 默认除法得到浮点数

## 打印函数

`from __future__ import print_function`

- 统一使用 print 函数
