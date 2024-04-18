---
layout:     post   			
title:      python-filelock
subtitle:   
date:       2023-11-18
author:     zlw
header-img: 
catalog: 
tags:
    - work
---

[toc]

### filelock

参考博客：

[python filelock介绍 - mlllily - 博客园 (cnblogs.com)](https://www.cnblogs.com/mlllily/p/17732910.html)

### 一、定义

FileLock是一个跨平台（windows、linux）的文件锁机制，常用于在多线程或多进程环境下，控制对共享资源（例如文件）的访问。

在python中，filelock包提供了FileLock类，可用于创建文件锁。这个类提供了一些方法来获取和释放文件锁，如acquire()和release()。

### 二、安装与使用

`pip install filelock`

`form filelocl import FileLock`

### 三、背景

项目中将远程文件请求下来后写入到本地环境

![](E:\NOTE!!!!\zlw1115.github.io\img\2023-11-18\filelock-1.jpg)

### 四、初步了解

```
from filelock import FileLock

lock = FileLock("my_lock_file.lock")
with lock: # 使用上下文管理器（with）可以自动获取和释放锁
    # 访问共享资源
    pass
```

### 五、知识点明晰

模拟写入数据到一个文件，由于可能在并行或并发环境下工作，因此需要确保在一个时间点只有一个进程或线程在写入文件，以防止数据损坏。这就是文件锁filelock发挥作用的地方，使用它来确保在写入文件时获取到锁，并在写入完成后释放锁。

```
from filelock import FileLock

file_path = "shared_file.txt"
lock_path = "shared_file.lock"

lock = FileLock(lock_path, timeout=1)

try:
    with lock:
        # 确保在写入文件时获取锁
        with open(file_path, "a") as file:
            file.write("Hello\n")
            print("successfully wrote to the file")
excep Timeout:
    print("another process is alreadly writing to the file")
```

在这个例子中，使用FileLock创建了一个文件锁，并设置了一个超时时间。当这个锁被其他进程占用时，程序将会阻塞直到超时，with lock语句确保获取和释放锁偶会正确执行，无论在这个代码块是否引发异常。

### 六、应用