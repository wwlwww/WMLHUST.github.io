---
title: Map并发安全实现原理
tags: [数据结构]
categories: [数据结构]
comments: true
date: 2020-05-14 11:03:32
updated: 2020-05-14 11:03:32
---

### Java Concurrent hashmap
1. 多个segment，支持最大segment数量的并发访问
> ps: 如果hash桶的list过长，可以使用红黑树代替list

### golang sync.Map
1. read-only, dirty 两个字段将读写分离
2. read-only不需加锁，读或写dirty都需要加锁
3. misses字段，统计read-only穿透次数，超过一定次数将dirty同步到read-only上
4. 删除时，通过给read-only添加标记，延迟删除
5. 读的时候，先查询read，不存在时查询dirty；写入时则只写入dirty
6. 写入过程，每次写入时，先copy 未删除的read-only到dirty中，然后将k-v存入dirty。
    > read-only可以当做dirty的缓存。dirty里的数据，总比read-only的多。
5. **适用于读多写少的场景。写入较多时，性能无法保证。**