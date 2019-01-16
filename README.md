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
