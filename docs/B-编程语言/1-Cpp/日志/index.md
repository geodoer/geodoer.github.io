---
title: C++日志系统
---

## 日志系统
「重要性」

1.  日志系统几乎是每一个实际的软件项目从开发、测试到交付，再到后期的维护过程中极为重要的查看软件代码运行流程、还原错误现场、记录运行错误位置及上下文等的重要依据。
2.  一个高性能的日志系统，能够准确记录重要的变量信息，同时又没有冗余的打印导致日志文件记录无效的数据。

## C++日志

| 库 | 说明 |
| --- | --- |
| log4c | 纯C的东西，移植性强。不再有人维护了。不是面向对象的，不支持流式log输入。有配置文件。最新版本（log4c-1.2.4.tar.gz）存在内存泄露。不建议使用 |
| log4cplus | 简洁, 下载的包编译顺利, 测试例子也能顺利运行 |
| [log4cxx](https://logging.apache.org/log4cxx/latest_stable/) | 臃肿, 需要引用apr(Apache Portable Runtime), 最痛苦的是老是编译不了 |
| log4cpp | 落后, 最后更新于2017年，编译成功后，使用有异常问题 |


# 相关链接

1.  [C++设计实现日志系统](https://blog.csdn.net/sinat_21107433/article/details/103102542)
2.  [一个著名的日志系统是怎么设计出来的？](https://mp.weixin.qq.com/s/XiCky-Z8-n4vqItJVHjDIg)
3.  [C/C++log日志库比较](https://blog.csdn.net/gatieme/article/details/50603682)
