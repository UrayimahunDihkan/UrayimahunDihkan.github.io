---
title: 关于CPU缓存
date: 2024-11-14 10:12:42
tags: tech
---

> cpu与内存简易关系

![img](https://pica.zhimg.com/v2-7a6b29e7cca6a4f5ae665c543ad9f9d0_1440w.jpg)

[计算机组成原理](https://zhida.zhihu.com/search?content_id=240951890&content_type=Article&match_order=1&q=计算机组成原理&zhida_source=entity)中，也可以简单认为cpu和内存是直接通信的，程序在运行时，将内存条中的01010010...指令读到cpu寄存器里寄存以及由ALU来计算，返回的结果写到寄存器或者写回到内存条里，成千上万个运算实现了代码的运行逻辑。

不过，真正实际的[嵌入式设备](https://zhida.zhihu.com/search?content_id=240951890&content_type=Article&match_order=1&q=嵌入式设备&zhida_source=entity)或电脑主板上的内存和cpu中间还隔着缓存和[总线](https://zhida.zhihu.com/search?content_id=240951890&content_type=Article&match_order=1&q=总线&zhida_source=entity)(缓存在cpu芯片内部)。以4核cpu为例，像这样：

![img](https://pic1.zhimg.com/v2-c0094257ffe8173fd24d472a72ec8f40_1440w.jpg)

L1是一级缓存，L2是二级缓存，L3是三级缓存，再到内存条。离cpu越近的读写越快，L1,L2,L3的一次IO速度大约是1ns,3ns,15ns。内存条的IO速度是60ns至80ns，再加上大约20ns的总线传输速度，cpu跟内存条之间起码需要80ns-100ns，跟L1,L2,L3缓存相比，可以说非常的拉胯。（这里的时间的数据是从美团技术团队的一个文章里面读到的留的印象，跟cpu的代数，架构，性能等硬件系数有关，不是绝对的）

[缓存技术](https://zhida.zhihu.com/search?content_id=240951890&content_type=Article&match_order=1&q=缓存技术&zhida_source=entity)可以很明显的提高机器的运行效率。记得大三的时候学习嵌入式系统移植，[jz2440](https://zhida.zhihu.com/search?content_id=240951890&content_type=Article&match_order=1&q=jz2440&zhida_source=entity)从启动boot，加载linux内核，加载文件系统等，一开始大概花了15秒左右，当我们根据jz2440的芯片手册打开了它的 [I/D cache](https://zhida.zhihu.com/search?content_id=240951890&content_type=Article&match_order=1&q=I%2FD+cache&zhida_source=entity)后启动时间降至5秒。

原因在于，程序在运行的时候，cpu会频繁访问某些变量，比如a++一百万次的代码访问a很频繁，激烈并发中cas操作读取AQS中的state变量也很频繁..., 在这种场景，如果有缓存的夹持，则CPU访问一个变量是先看一级缓存有没有，没有看二级，三级，还没有再看主内存，频繁变量经常出现在L1,L2,L3等缓存中，CPU可以1ns,3ns,15ns级别的读取它们，没必要耗那么多时间去内存条读，所以会快。

[缓存行](https://zhida.zhihu.com/search?content_id=240951890&content_type=Article&match_order=1&q=缓存行&zhida_source=entity)是L1,L2,L3的缓存最小单位，大小为64Byte. 相当于long[8]，八个长度的long数组。

相邻的两个数据有极大的可能被cpu交替访问，或者说，前几个ns里cpu访问了内存中0xFFFF0123地址的a变量，紧接着过1ns后cpu又想访问0xFFFF0124地址的b变量，那么这个时候CPU需要从内存条中读取b吗？

大概率不用。因为缓存机制，极有可能将a,b同时放在了64Byte的缓存行里缓存起来了，b可以直接在缓存中读取。缓存数据范围是这样的：CPU想读a，CPU不会只缓存a，还会把a后面相邻的b,c,d...等也缓存进来刚好要凑满一个缓存行的大小。原因还是计算机在执行代码的时候的访问变量的特征，访问a紧接着还会访问隔壁的b，隔壁的c，索性只要访问第一个，那把其他几个也填满缓存行里，使在cpu最近的位置最快的获取b,c,d...



### demo:

```java
public class CacheLineTest {

    static long[][] arr;
    public static void main(String[] args) {

        /* 按64Byte装一个二维数组，每次读取64Byte子数组 充分利用cacheLine */
        arr = new long[1024 * 1024][];
        for (int i = 0; i < 1024 * 1024; i++) {
            arr[i] = new long[8];
            for (int j = 0; j < 8; j++) {
                arr[i][j] = 0L;
            }
        }

        long sum = 0L;
        long marked = System.currentTimeMillis();
        for (int i = 0; i < 1024 * 1024; i++) {
            for (int j = 0; j < 8; j++) {
                sum = arr[i][j];
            }
        }

        System.out.println("Loop time: " + (System.currentTimeMillis() - marked) + "ms");

      
          /* 故意不按相邻变量访问 */
        marked = System.currentTimeMillis();
        for (int i = 0; i < 8; i++) {
            for (int j = 0; j < 1024 * 1024; j++) {
                sum = arr[j][i];
            }
        }
        System.out.println("Loop time: " + (System.currentTimeMillis() - marked) + "ms");
    }
}


//结果
Loop time: 24ms
Loop time: 64ms
```



AQS源码中的state为什么要volatile修饰的用意在这里。

CPU缓存的应用，是平庸和非凡工程作品的分水岭，只有master才能透视计算机内脏，掌握这里的“火候”。
