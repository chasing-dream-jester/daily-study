# Redis 进阶
## Redis 事务
Redis 事务提供了一种将多个命令打包成一个原子操作的机制。在事务中，所有命令要么全部执行，要么全部不执行。Redis 的事务通过 `MULTI`、`EXEC`、`DISCARD` 和 `WATCH` 命令来管理。

### 事务命令

1. **MULTI**
   开始一个事务。之后的所有命令将被放入事务队列，直到执行 `EXEC` 命令。

2. **EXEC**
   执行事务中的所有命令。

3. **DISCARD**
   取消事务，放弃事务中的所有命令。

4. **WATCH key [key ...]**
   监视一个或多个键，如果在事务执行之前这些键被修改，事务将被中止。

5. **UNWATCH**
   取消对所有键的监视。

### 使用示例

#### 基本事务

以下示例展示了一个基本的事务，包括开始事务、添加命令和执行事务：

```sh
# 开始事务
MULTI

# 添加命令到事务
SET key1 "value1"
SET key2 "value2"

# 执行事务
EXEC
```

#### 取消事务

如果在事务中需要放弃所有命令，可以使用 `DISCARD` 命令：

```sh
# 开始事务
MULTI

# 添加命令到事务
SET key1 "value1"
SET key2 "value2"

# 取消事务
DISCARD
```

#### 使用 WATCH 实现乐观锁

`WATCH` 命令用于监视一个或多个键，如果在事务执行之前这些键被修改，事务将被中止。这种机制可以用于实现乐观锁。

以下示例展示了如何使用 `WATCH` 命令实现乐观锁：

```sh
# 监视键
WATCH mykey

# 获取当前值
value=$(redis-cli GET mykey)

# 开始事务
MULTI

# 设置新值
redis-cli SET mykey "new_value"

# 执行事务
EXEC
```

如果在 `WATCH` 和 `EXEC` 之间 `mykey` 被其他客户端修改，事务将被中止，`EXEC` 返回空响应（`nil`）。

### 示例：转账操作

以下示例展示了如何使用 Redis 事务实现一个简单的转账操作：

```sh
# 监视账户余额
WATCH account:1 balance

# 获取当前余额
balance=$(redis-cli GET account:1 balance)

# 检查余额是否足够
if [ $balance -lt $amount ]; then
  echo "Insufficient funds"
  redis-cli UNWATCH
else
  # 开始事务
  redis-cli MULTI

  # 扣减账户余额
  redis-cli DECRBY account:1 balance $amount

  # 增加目标账户余额
  redis-cli INCRBY account:2 balance $amount

  # 执行事务
  redis-cli EXEC
fi
```

在这个示例中，如果在 `WATCH` 和 `EXEC` 之间 `account:1 balance` 被其他客户端修改，事务将被中止，`EXEC` 返回空响应（`nil`）。

### 注意事项

- **事务中的命令不会立即执行**：在事务中，所有命令都会被放入事务队列，直到执行 `EXEC` 命令时才会执行。
- **事务中的命令不会部分执行**：在事务中，所有命令要么全部执行，要么全部不执行。
- **事务中的命令不会回滚**：如果事务中的某个命令失败，其余命令仍会继续执行。Redis 事务没有回滚机制。
- **WATCH 和 UNWATCH**：`WATCH` 命令用于监视键的变化，如果在事务执行之前这些键被修改，事务将被中止。`UNWATCH` 命令用于取消对所有键的监视。

通过合理使用 Redis 的事务命令，可以确保多个命令在一个原子操作中执行，从而实现数据一致性和并发控制。


## Redis 发布订阅
Redis 的发布/订阅（Publish/Subscribe，简称 Pub/Sub）功能是一种消息传递模式，允许发送者（发布者）将消息发送到一个频道（channel），并允许接收者（订阅者）订阅这些频道以接收消息。发布者和订阅者之间是松散耦合的，这使得 Pub/Sub 模式非常适合实现实时消息传递、事件通知、日志收集等场景。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系

![subscribe](../images/pubsub1.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端

![publish](../images/pubsub2.png)


### 主要命令

1. **PUBLISH channel message**
   将消息发布到指定的频道。
   ```sh
   PUBLISH mychannel "Hello, World!"
   ```

2. **SUBSCRIBE channel [channel ...]**
   订阅一个或多个频道，开始接收这些频道的消息。
   ```sh
   SUBSCRIBE mychannel
   ```

3. **UNSUBSCRIBE [channel [channel ...]]**
   退订一个或多个频道。如果没有指定频道，则退订所有频道。
   ```sh
   UNSUBSCRIBE mychannel
   ```

4. **PSUBSCRIBE pattern [pattern ...]**
   订阅与给定模式匹配的所有频道。
   ```sh
   PSUBSCRIBE my*
   ```

5. **PUNSUBSCRIBE [pattern [pattern ...]]**
   退订与给定模式匹配的所有频道。如果没有指定模式，则退订所有模式。
   ```sh
   PUNSUBSCRIBE my*
   ```

### 示例

以下是一个使用 Redis 发布/订阅功能的完整示例，包括发布者和订阅者。

#### 订阅者

订阅者订阅一个或多个频道以接收消息：

```sh
# 订阅频道 mychannel
SUBSCRIBE mychannel
```

输出：

```sh
1) "subscribe"
2) "mychannel"
3) (integer) 1
```

#### 发布者

发布者将消息发布到指定的频道：

```sh
# 将消息发布到频道 mychannel
PUBLISH mychannel "Hello, World!"
```

输出：

```sh
(integer) 1
```

订阅者接收到的消息：

```sh
1) "message"
2) "mychannel"
3) "Hello, World!"
```

#### 使用模式订阅

订阅者可以使用模式订阅与给定模式匹配的所有频道：

```sh
# 订阅与 my* 匹配的所有频道
PSUBSCRIBE my*
```

输出：

```sh
1) "psubscribe"
2) "my*"
3) (integer) 1
```

发布者将消息发布到匹配模式的频道：

```sh
# 将消息发布到频道 mychannel1
PUBLISH mychannel1 "Hello, Channel 1!"
```

订阅者接收到的消息：

```sh
1) "pmessage"
2) "my*"
3) "mychannel1"
4) "Hello, Channel 1!"
```

### 完整示例

以下是一个完整的示例，包括多个订阅者和发布者：

#### 订阅者 1

```sh
# 订阅频道 mychannel
SUBSCRIBE mychannel
```

#### 订阅者 2

```sh
# 订阅频道 mychannel 和 mychannel2
SUBSCRIBE mychannel mychannel2
```

#### 发布者

```sh
# 将消息发布到频道 mychannel
PUBLISH mychannel "Hello, World!"

# 将消息发布到频道 mychannel2
PUBLISH mychannel2 "Hello, Channel 2!"
```

#### 订阅者 1 接收到的消息

```sh
1) "message"
2) "mychannel"
3) "Hello, World!"
```

#### 订阅者 2 接收到的消息

```sh
1) "message"
2) "mychannel"
3) "Hello, World!"
1) "message"
2) "mychannel2"
3) "Hello, Channel 2!"
```

### 注意事项

- **消息丢失**：如果订阅者在消息发布时没有订阅相应的频道，则会错过该消息。Redis Pub/Sub 不保证消息的持久化。
- **性能**：Redis 的 Pub/Sub 模式适用于实时消息传递，但如果消息量非常大或订阅者数量非常多，可能会影响 Redis 的性能。
- **消息顺序**：在 Redis 的 Pub/Sub 模式中，消息是按发送顺序传递的，但如果订阅者处理消息的速度较慢，可能会导致消息积压。

### 使用场景

- **实时通知**：适用于实时事件通知，例如聊天消息、系统报警等。
- **日志收集**：可以将日志消息发布到频道，多个订阅者可以同时接收并处理日志。
- **分布式系统**：在分布式系统中，可以使用 Pub/Sub 模式实现节点间的消息传递和协调。

通过合理使用 Redis 的发布/订阅功能，可以实现高效的实时消息传递，满足各种实时通信需求。
