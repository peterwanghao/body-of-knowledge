#  RabbitMQ

## 1. Basic.Reject 的用法是什么？

答：该信令可用于 consumer 对收到的 message 进行 reject 。若在该信令中设置 requeue=true，则当 RabbitMQ server 收到该拒绝信令后，会将该message 重新发送到下一个处于 consume 状态的 consumer 处（理论上仍可能将该消息发送给当前 consumer）。若设置 requeue=false ，则RabbitMQ server 在收到拒绝信令后，将直接将该 message 从 queue 中移除。



另外一种移除 queue 中 message 的小技巧是，consumer 回复 Basic.Ack但不对获取到的 message 做任何处理。而 Basic.Nack 是对 Basic.Reject 的扩展，以支持一次拒绝多条 message的能力。



## 2.为什么不应该对所有的 message 都使用持久化机制？

答：首先，必然导致性能的下降，因为写磁盘比写 RAM 慢的多，message的吞吐量可能有 10 倍的差距。

其次，message 的持久化机制用在RabbitMQ 的内置 cluster 方案时会出现“坑爹”问题。矛盾点在于，若message 设置了 persistent 属性，但 queue 未设置 durable 属性，那么当该 queue 的 owner node 出现异常后，在未重建该 queue 前，发往该queue 的 message 将被 blackholed ；若 message 设置了 persistent属性，同时 queue 也设置了 durable 属性，那么当 queue 的 ownernode 异常且无法重启的情况下，则该 queue 无法在其他 node 上重建，只能等待其 owner node 重启后，才能恢复该 queue 的使用，而在这段时间内发送该 queue 的 message 将被 blackholed 。

所以，是否要对message 进行持久化，需要综合考虑性能需要，以及可能遇到的问题。若想达到 100,000 条/秒以上的消息吞吐量（单 RabbitMQ 服务器），则要么使用其他的方式来确保 message 的可靠 delivery ，要么使用非常快速的存储系统以支持全持久化（例如使用 SSD）。

另外一种处理原则是：仅对关键消息作持久化处理（根据业务重要程度），且应该保证关键消息的量不会导致性能瓶颈。

## 3.为什么 heavy RPC 的使用场景下不建议采用 disk node ？

答：heavy RPC 是指在业务逻辑中高频调用 RabbitMQ 提供的 RPC 机制，导致不断创建、销毁 reply queue ，进而造成 disk node 的性能问题（因为会针对元数据不断写盘）。所以在使用 RPC 机制时需要考虑自身的业务场景。

## 4.向不存在的 exchange 发 publish 消息会发生什么？向不存在的queue 执行 consume 动作会发生什么？

答：都会收到 Channel.Close 信令告之不存在（内含原因 404NOT_FOUND）。

## 5.什么情况下 producer 不主动创建 queue 是安全的？

答：1.message 是允许丢失的；2.实现了针对未处理消息的 republish 功能（例如采用 Publisher Confirm 机制）。

## 6.“dead letter”queue 的用途？

答：当消息被 RabbitMQ server 投递到 consumer 后，但 consumer 却通过 Basic.Reject 进行了拒绝时（同时设置 requeue=false），那么该消息会被放入“dead letter”queue 中。该 queue 可用于排查 message 被reject 或 undeliver 的原因。

## 7.为什么说保证 message 被可靠持久化的条件是 queue 和 exchange具有 durable 属性，同时 message 具有 persistent 属性才行？

答：binding 关系可以表示为 exchange – binding – queue 。从文档中我们知道，若要求投递的 message 能够不丢失，要求 message 本身设置persistent 属性，要求 exchange 和 queue 都设置 durable 属性。其实这问题可以这么想，若 exchange 或 queue 未设置 durable 属性，则在其crash 之后就会无法恢复，那么即使 message 设置了 persistent 属性，仍然存在 message 虽然能恢复但却无处容身的问题；同理，若 message 本身未设置 persistent 属性，则 message 的持久化更无从谈起。