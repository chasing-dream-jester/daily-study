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


## Redis分布式锁
在分布式系统中，分布式锁是一种确保多个进程或节点不会同时访问共享资源的机制。Redis 提供了一种简单而有效的方式来实现分布式锁。通过使用 Redis 的 `SET` 命令和一些特定的参数，可以实现一个基本的分布式锁。

### 使用 Redis 实现分布式锁

#### 1. 获取锁

使用 `SET` 命令设置一个键，并通过 `NX` 和 `EX` 参数确保原子性和超时机制：

- `NX`：仅当键不存在时设置键。
- `EX`：设置键的过期时间（单位为秒）。

```sh
SET lock_key unique_value NX EX 10
```

如果返回 `OK`，表示成功获取锁；否则，表示锁已被其他客户端持有。

#### 2. 释放锁

使用 `DEL` 命令删除锁。为了确保锁的持有者才可以释放锁，需要使用 Lua 脚本进行原子检查和删除操作。

```lua
if redis.call("GET", KEYS[1]) == ARGV[1] then
    return redis.call("DEL", KEYS[1])
else
    return 0
end
```

### 示例实现

下面是一个使用 Node.js 实现 Redis 分布式锁的示例：

#### 安装依赖

首先，安装 `ioredis` 依赖：

```sh
npm install ioredis
```

#### 获取锁和释放锁的函数

```javascript
const Redis = require('ioredis');
const redis = new Redis();

// 获取锁
async function acquireLock(key, value, ttl) {
  const result = await redis.set(key, value, 'NX', 'EX', ttl);
  return result === 'OK';
}

// 释放锁
async function releaseLock(key, value) {
  const script = `
    if redis.call("GET", KEYS[1]) == ARGV[1] then
      return redis.call("DEL", KEYS[1])
    else
      return 0
    end
  `;
  const result = await redis.eval(script, 1, key, value);
  return result === 1;
}

// 使用锁的示例
async function main() {
  const lockKey = 'lock_key';
  const lockValue = 'unique_value';
  const ttl = 10; // 锁的过期时间为 10 秒

  // 尝试获取锁
  const isLocked = await acquireLock(lockKey, lockValue, ttl);
  if (isLocked) {
    console.log('锁已获取');

    // 在锁持有期间执行一些操作
    // ...

    // 释放锁
    const isReleased = await releaseLock(lockKey, lockValue);
    if (isReleased) {
      console.log('锁已释放');
    } else {
      console.log('释放锁失败');
    }
  } else {
    console.log('获取锁失败');
  }

  redis.quit();
}

main();
```

### 详细解释

1. **获取锁 `acquireLock`**：
   - 使用 `SET` 命令设置锁键，带有 `NX` 和 `EX` 参数。
   - `NX` 参数确保仅当键不存在时设置锁键。
   - `EX` 参数设置锁键的过期时间，防止死锁。

2. **释放锁 `releaseLock`**：
   - 使用 Lua 脚本进行原子检查和删除操作。
   - 脚本检查锁键的值是否与提供的值匹配，如果匹配则删除锁键。

3. **使用锁 `main`**：
   - 尝试获取锁，如果成功则执行一些操作，并在完成后释放锁。
   - 如果获取锁失败，可以选择重试或退出。

### 注意事项

- **唯一值**：确保每个客户端使用唯一的值来标识持有的锁。可以使用 UUID 或其他唯一标识符。
- **超时机制**：设置合理的锁过期时间，防止死锁。如果操作需要长时间完成，可以考虑使用锁续租机制。
- **高可用性**：在高可用场景中，可以使用 Redis 集群或 Redis Sentinel 来确保 Redis 服务的高可用性。

### 使用 Redlock 算法

对于更高级的分布式锁实现，可以使用 Redlock 算法。Redlock 是由 Redis 作者提出的一种分布式锁算法，适用于 Redis 集群环境。可以使用 `node-redlock` 库实现 Redlock 算法：

```sh
npm install redlock
```

使用示例：

```javascript
const Redis = require('ioredis');
const Redlock = require('redlock');

const redis = new Redis();
const redlock = new Redlock(
  [redis],
  {
    driftFactor: 0.01, // 时钟漂移因子
    retryCount: 10, // 重试次数
    retryDelay: 200, // 重试延迟
    retryJitter: 200 // 随机延迟
  }
);

async function main() {
  try {
    // 获取锁
    const lock = await redlock.lock('lock_key', 10000); // 锁过期时间为 10 秒
    console.log('锁已获取');

    // 在锁持有期间执行一些操作
    // ...

    // 释放锁
    await lock.unlock();
    console.log('锁已释放');
  } catch (err) {
    console.error('获取锁失败', err);
  }

  redis.quit();
}

main();
```

通过合理使用 Redis 的分布式锁机制，可以确保多个进程或节点在分布式环境中安全地访问共享资源。

### QA
为什么不直接使用 redis.del(lockKey)来释放锁

为了确保只有持有锁的客户端才能释放锁，通常需要在释放锁时进行原子检查。这就是为什么我们使用 Lua 脚本来实现这一操作。Lua 脚本在 Redis 中是原子执行的，可以确保检查和删除操作在同一个事务中完成。

直接使用 redis.del(lockKey) 存在以下问题：

- 锁的持有者不一致 ：如果一个客户端在持有锁期间崩溃或超时，另一个客户端可能会获取到相同的锁键，并在执行 DEL 命令时意外地删除了另一个客户端的锁。
- 数据竞争 ：在高并发场景下，多个客户端可能会同时尝试删除锁，导致数据竞争问题。
- 锁的安全性 ：为了确保只有持有锁的客户端才能释放锁，需要在删除锁之前进行原子检查。

以下是 Lua 脚本的实现，它确保只有当锁的值与客户端的唯一标识符匹配时，才会删除锁：
```js
if redis.call("GET", KEYS[1]) == ARGV[1] then
    return redis.call("DEL", KEYS[1])
else
    return 0
end
```

### demo
使用redis锁来解决token或者cookie过期使用refreshToken去刷新Token竞争性问题。
```js
// 释放锁
async function releaseLock(key, value) {
  const script = `
    if redis.call("GET", KEYS[1]) == ARGV[1] then
      return redis.call("DEL", KEYS[1])
    else
      return 0
    end
  `;
  const result = await redis.eval(script, 1, key, value);
  return result === 1;
}
// 过期时间为15s，与接口超时时间保持一致，重试次数50次（值为timeout/delay）
async function setKeyWithLock(callback: () => void, key: string, redis: IORedis.Redis | IORedis.Cluster, timeout: number = 15, retries: number = 50) {
    const lockKey = `lock:${key}`;
    const lockValue = new Date().getTime();

    for (let i = 0; i < retries; i++) {
      // 生成锁
      const multi = redis.multi();
      multi.setnx(lockKey, lockValue);
      multi.expire(lockKey, timeout);
      const [acquiredLock] = await multi.exec();

      if (acquiredLock?.[1] === 1) {
        try {
            await callback();
            return;
        } finally {
            // 释放锁
            await releaseLock(lockKey, lockValue)
            // await redis.del(lockKey);
        }
      } else {
        // 未能获取锁，等0.3s后重试
        await new Promise(resolve => setTimeout(resolve, 300));
      }
    }

    throw new Error(`Failed to acquire lock after ${retries} retries`);
  }
```
