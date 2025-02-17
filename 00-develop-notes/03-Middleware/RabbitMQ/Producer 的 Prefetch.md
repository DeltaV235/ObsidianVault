---

title: Producer 的 Prefetch
created: 2025-02-17
tags:
    - Java
    - SpringBoot
    - RabbitMQ
---

## Prefetch 的作用

消费端可以在 application.yml 中配置 prefetchCount，表示每次从队列中取多少条消息。

- 在 RabbitMQ 中，prefetch 是每个消费者（Consumer）的消息预取计数限制，表示 RabbitMQ 会在消费者确认（ack）之前最多发送多少条未确认的消息给该消费者。
- 如果 listener 配置使用了 simple 模式，并且设置了多个并发消费者（通过 concurrency 配置），那么 prefetch 是对每个消费者线程分别生效的。

**例如：**

- 如果 prefetch 设置为 10，并且 concurrency 设置为 5，则总共会预取 10 × 5 = 50 条消息。
- 每个线程（消费者）最多同时处理 10 条消息。

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 10
        concurrency: 5
```

**注意：**

- 在 自动 ACK 模式 (acknowledge-mode: auto) 下，prefetch 的消息并不会在预取时自动 ACK，而是会在消息被实际传递到监听器（listener）并进入处理逻辑时被自动 ACK。

## Prefetch 的默认值

- 默认情况下，RabbitMQ 的 prefetchCount 为 250。
