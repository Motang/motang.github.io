---
layout: post
title: CMS之promotion failed&concurrent mode failure
category: java jvm
tags: [java jvm CMS GC]
comments: true
share: true
---
## 目录 ##

* Table of Contents
{:toc}

## 介绍 ##
CMS并行GC收集器是大多数JAVA服务应用的最佳选择，然而，CMS并不是完美的，在使用CMS的过程中会产生2个最让人头痛的问题：

> promotion failed 

> concurrent mode failure

## 原因和解决思路 ##

promotion failed 是在进行Minor GC时，survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的(promotion failed时老年代CMS还没有机会进行回收，又放不下转移到老年代的对象，因此会出现下一个问题concurrent mode failure)

> 方法一：调大新生代或者Survivor空间（survivor空间可以放置大对象几率增大，减少对象直接进入旧生代的机会，用minor GC策略来回收临时大对象）

> 方法二：调低CMS GC的触发阀值，调高full gc时压缩的频次（旧生代可提前CMS GC，调高压缩频次是采用标记整理清除算法回收内存，避免内存碎片，降低了大对象内存分配不足的几率）


concurrent mode failure 出现此现象的原因主要有两个：一个是在年老代被用完之前不能完成对无引用对象的回收；一个是当新空间分配请求在年老代的剩余空间中得不到满足。造成concurrent mode failure的原因是当minor GC进行时，1）旧生代所剩下的空间小于Eden区域+From区域的空间，或者2）在CMS执行老年代的回收时有业务线程试图将大的对象放入老年代，导致CMS在老年代的回收慢于业务对象对老年代内存的分配。

> 方法一：是调低CMS GC的触发阀值，即参数-XX:CMSInitiatingOccupancyFraction的值，默认值是68，可以调低，让CMS GC尽早执行，以保证有足够的空间，调高full gc时压缩的频次

> 方法二：即减少年轻代大小，避免放入年老代时需要分配大的空间，同时调高full gc时压缩碎片的频次，减少持久代大小，以及将适当调大CMS GC的触发阀值（因为年轻代小了，这个调大点没关系，后面可以再调试这个参数）

> 方法三：增加堆的空间大小

## 总结 ##

CMS GC出现上述两种问题，也提供了的解决方法调优的思路，需要多次调整参数后，调整参数后一定需要观察和监控GC的日志，出现问题，及时可通知到相关负责人，直到调整到最适当此应用的JVM参数。

因此，采用CMS并行GC的应用，尤其有大量用户的互联网应用，调整到适当的JVM参数，需要多次的实践和验证，每一种应用JVM参数都可能不一样的。

