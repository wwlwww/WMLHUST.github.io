---
title: kafka关键原理.md
tags: [kafka,消息队列]
categories: [后端,系统原理]
comments: true
date: 2020-09-27 16:35:52
updated: 2020-09-27 16:35:52
description:
---
1. kafka如何保证高吞吐量  
    - Partition机制：同一个topic的partition分布在不同的节点上，可以通过增加partition的数量，线性扩展topic的读/写吞吐量。  
    - 消息压缩：CPU消耗与压缩比的权衡压缩算法。
    - 0拷贝：
    - 批量发送/读取：batch.size/linger.ms，kafka会在缓存消息达到size大小或者时间达到linger.ms时发送。消费时同样有对应的设置，fetch.min.bytes/fetch.max.wait.ms。

2. kafka如何保证数据可靠，不丢失
    - ack机制：将生产者的ack设为all
    - min.insync.replicas，该配置项的默认值是1，既在acks=all时，最少得有一个Replica进行确认回执。建议在生产环境配置为2，保证数据的完整性。

3. kafka能否保证消息的有序性  
   - 在topic的同一个partition内可以，在partition之间不可以。
   - max.in.flight.requests.per.connection = 1，表示客户端在单个连接上能够发送的未响应请求的个数，设为1就表示，在同一时刻只能有一个未响应请求。

4. kafka的消息数据是怎么存的

5. kafka集群如果一个节点宕机，会发生什么

6. kafka的ack机制  
    - ack=0: 生产者不需等待服务器响应。甚至client会使用批量机制，压根就没发请求。但是它可以支持最高的吞吐量。
    - ack=1: 只要partition的leader节点收到消息，就返回成功。但如果leader挂掉，而该消息还未同步给其他follower，则消息依然会丢失。
    - ack=all: 当partition的所有结点都收到消息，才返回成功。数据可靠性最好，但是broker的延迟最大。
    - 
7. 再均衡 Rebalance   
消费者消费分区发生变化，会触发一次再均衡。在再均衡期间，消费者无法读取消息，会造成整个群组一小段时间的不可用。  
消费者和kafka之间保持有心跳，如果消费者停止发送心跳，会话就会过期，群组协调器就认为它已经死亡，就会触发一次再均衡。  
每个topic，会选择一个broker充当group coordinator角色，由它来存储group相关meta信息。  
为了避免不必要的rebalance：
    - 增大session.timeout.ms，相当于增加会话超时时间，缺点是节点崩溃检测时间会更长，因此对应分区的消息消费可能会延迟。
    - 增大max.poll.interval.ms，避免consumer消费时间过长，导致rebalance。  

    分区分配策略：  
    第一个加入群组的消费者会成为leader consumer，它从group coordinator获取消费者列表，然后给每个消费者分配分区。然后将分配结果发回给group coordinator，最终发给每个消费者，每个消费者只能看到自己分配到的分区。

8. offset持久化  
旧版本的kafka使用zookeeper保存。   
新版本的kafka自己保存，消费者往_consumer_offset这个特殊topic发送消息，消息里包含了分区的偏移量。   

9. offset自动提交  
如果enable.auto.commit设为true，那么每过5s，会将pull到的消息里，最大的偏移量commit上去。虽然方便，但是如果消费者发生故障，可能会导致消息遗漏和大量消息重复处理，因为无法确认当前offset之前的消息都已经成功处理。

10.   
