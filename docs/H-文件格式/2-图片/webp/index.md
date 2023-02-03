---
title: WebP
---

WebP是一种同时提供了有损压缩与无损压缩（可逆压缩）的图片文件格式。

相关链接

1. [WebP API 文档  |  Google Developers](https://developers.google.com/speed/webp/docs/api?hl=zh-cn)
2. [编译实用程序  |  WebP  |  Google Developers](https://developers.google.com/speed/webp/docs/compiling?hl=zh-cn)
3. [使用入门  |  WebP  |  Google Developers](https://developers.google.com/speed/webp/docs/using?hl=zh-cn)

## 简介

> 来源：[WebP - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/WebP)

### 历史
WebP最初在2010年9月发布，其支持库于2018年4月发布1.0版本。截至2021年5月，已有94%的浏览器支持此格式。

### 设计目的
WebP的设计目标是在减少文件大小的同时，达到和[JPEG](https://zh.wikipedia.org/wiki/JPEG "JPEG")、[PNG](https://zh.wikipedia.org/wiki/PNG "PNG")、[GIF](https://zh.wikipedia.org/wiki/GIF "GIF")格式相同的图片质量，并希望借此能够减少图片档在网络上的发送时间。

### 性能
根据Google较早的测试，WebP的无损压缩比网络上找到的[PNG](https://zh.wikipedia.org/wiki/PNG "PNG")档少了45％的文件大小，即使这些PNG档在使用[pngcrush](https://zh.wikipedia.org/w/index.php?title=Pngcrush&action=edit&redlink=1 "Pngcrush（页面不存在）")和[PNGOUT](https://zh.wikipedia.org/w/index.php?title=PNGOUT&action=edit&redlink=1 "PNGOUT（页面不存在）")处理过，WebP还是可以减少28％的文件大小。

WebP支持的像素最大数量是16383x16383。

- 有损压缩的WebP仅支持8-bit的YUV 4:2:0格式
- 而无损压缩（可逆压缩）的WebP支持VP8L编码与8-bit之RGBA色彩空间
- 而无论是有损或无损压缩皆支持Alpha透明通道、ICC色彩配置、XMP诠释资料。

## C++读写

WebP读写库下载地址（内含编译版本与源代码）：[Google WebP库](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/index.html)






