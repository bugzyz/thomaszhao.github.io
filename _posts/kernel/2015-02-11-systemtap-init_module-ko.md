---
title: 使用systemtap获取.ko文件
tagline: ""
category : kernel
layout: post
tags : [systemtap, ko, stap, xxd]
---
{% include JB/setup %}

有些产品是以内核模块`*.ko`形式存在的，使用`insmod`命令插入内核，
Linux同时也提供了`init_module(2)`函数。

前几天碰到一个友商的产品，将`.ko`文件隐藏在可执行文件中，没有直接提供明文的`.ko`文件，
在运行时调用`init_module`函数注册。

问题来了，怎么提取这种情况的`.ko`文件？答案当然是`systemtap`。

<!-- more -->

## 使用systemtap获取.ko文件

### 准备stp脚本：

root@thomas:systemtap# cat probe_init_module.stp

```
#!/usr/bin/stap -g -DMAXACTION=400000

probe begin
{
         log("Thomas: begin to probe")
}

probe syscall.init_module {
    printf ("%s(%d) init_module (%s)\n", execname(), pid(), argstr)
    printf ("count=%d\n", $len)

    for(i=0; i < $len; i++) {
     printf("%02x ", user_uint8($umod + i))
     if(i && i%16 == 0) {
         printf("\n")
     }
    }
    printf("\n")
    exit()
}

probe end
{
         log("Thomas: end to probe")
}
```

### 获取`.ko`文件

执行`stap`命令：

```
#/usr/bin/stap -g -DMAXACTION=400000  probe_init_module.stp -o 1.txt
```

执行会调用`init_module`的命令：

    # ./<xxx> start

使用`xxd`命令反向把hex变为文件：

    #xxd -r -p 1.txt <xxx>.ko

### 使用`strace`命令跟踪模块的参数

    # strace ./<xxx> start
    ## 查找其中`init_module`函数的参数

###使用insmod将模块插入：

    # insmod ./<xxx>.ko <other parameters>

大功高成~
下一步就打开`IDA pro`尽情的反吧。

