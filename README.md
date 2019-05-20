# blog
###### Java Thread 调度
###### Reactor
The reactor design pattern is an event handling pattern for handling service requests delivered concurrently to a service handler by one or more inputs.<br/>
反应器设计模式是能够将一个或者多个服务请求并发地派发给服务处理器的事件处理模式
###### JMX(java management extensions)
###### Spring
###### Kafka consume|producer|client
1. ConsumerCoordinator
 fetch|
 update consume position|
 period commit|
 CompletedFetch
 nextFetchOffset
 kafka 消费失败?

2. Kafka producer:
  RecordAccumulator
  Sender
  NetworkClient
3. Kafka Client
4. https://github.com/openpreserve/pagelyzer

https://github.com/edgi-govdata-archiving/awesome-website-change-monitoring/blob/master/readme.md

5. kafka producer

 ProdcuerRecord<K,V>
 key 路由字段
 value body

6. Kafka Metadata
异步请求
maybeUpdate
poll 中更新

7. Sender 后台发送线程|单线程发送
{
  KafkaClient(NetworkClient)
  accumulator(RecordAccumulator)
  sendProduceRequest
}
8. ApiKeys
  Type

9. AbstractRequest
10. NetworkClient/MetadataRequest

  Schema
  Field
  Struct
  serilize
  leadLoadNode
  DefaultMetadataUpdater
  NetworkSend
  inFlightRequests

11. PRODUCE流程
  ReadyCheckResult
12. 2PC &BASE transaction  
13. Kafka server
   KafkaApi handle
   Kafka acks
   Replicas 
   appendToLocalLog
   Partition   
   recvRegisterHandler
   ISR 
   Log
14. kafka isr(in-sync replicas)/failover
15. kafka benchmark boundary(large scale parition)
16. annotation 总结下
17. git reset&revert  https://es.atlassian.com/git/tutorials/undoing-changes/git-revert
18. https://github.com/jitsi/jitsi
19. two-phase transaction  and recovery
 - http://web.cs.iastate.edu/~cs554/NOTES/Ch8-4.pdf
20. AbstractProcessor 
21. BeanPostProcessor 
22. 字典压缩算法LZ4
23. HTTP 1.1 
https://tools.ietf.org/html/draft-ietf-httpbis-p2-semantics-16#section-7.5
24. ubuntu shadowsocks and terminal https://www.jianshu.com/p/48b3866b5e2a 
25. Linkerd RPC loadbalance
https://blog.linkerd.io/2016/03/16/beyond-round-robin-load-balancing-for-latency/#_ga=2.255177408.933373521.1558319908-1625069959.1558319908
