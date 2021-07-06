### Channels

`Deferred`提供了一种在协程之间传输单个值的便捷方法。`Channels`通道提供了一种传输 `stream` 的方法。

#### Channel 基础

`Channel` 在概念上与 `BlockingQueue` 非常相似。一个关键的区别是，它是一个挂起的发送操作，而不是阻塞的 `put` 操作，也不是阻塞的 `take` 操作，它有一个挂起的接收操作。

```kotlin
val channel = Channel<Int>()
launch {
    for (x in 1..5) channel.send(x)
}
repeat(5) { println(channel.receive()) }
println("Done")
```

#### 关闭和迭代 channels

与队列不同，`channel` 可以被关闭以指示不再有元素到来。在接收方，使用常规 `for` 循环从通道接收元素是很方便的。

从概念上讲，关闭就像向通道发送一个特殊的关闭令牌。一旦受到此关闭令牌，迭代就会停止，因此可以保证在关闭之前接收到所有之前发送的元素：

```kotlin
val channel = Channel<Int>()
launch {
    for (x in 1..5) channel.send(x * x)
    channel.close()
}
for (y in channel) println(y)
println("Done")
```

#### 建立 channel 生产商

协程生成元素序列的模式非常常见。这是生产者-消费者模式的一部分，经常出现在并发代码中。您可以将这样的生产者抽象为一个以 `channel` 作为参数的函数，但这与必须从函数返回结果的常识相反。

有一个方便的协程构建器，名为 `produce`，可以很容易地在生产者端正确地做到这一点，还有一个扩展函数 `consumeEach`，它取代了消费者端的 `for` 循环：

```kotlin
val squares = produceSquares()
squares.consumeEach { println(it) }
println("Done!")
```

#### 管道

管道是一种模式，其中一个协程可能产生无限的值：

```kotlin
fun CoroutineScope.produceNumbers() = produce {
    var x = 1
    while (true) send(x++)
}
```

另一个协程使用该流，进行一些处理，并产生一些其他结果：

```kotlin
fun CoroutineScope.square(numbers: ReceiveChannel<Int>): ReceiveChannel<Int> = produce {
    for (x in numbers) send(x * x)
}
```

`main` 函数连接整个管道：

```kotlin
val numbers = produceNumbers()
val squares = square(numbers)
repeat(5) {
    println(squares.receive())
}
println("Done!")
coroutineContext.cancelChildren()
```

所有创建协程的函数都被定义为 `CoroutineScope` 的扩展，这样我们就可以依靠结构化并发来确保我们的应用程序中没有全局协程而无法取消。

#### 带管道的素数

让我们通过一个使用协程管道生成素数的示例将管道发挥到极致。我们先从一个无限的数字序列开始。

```kotlin
fun CoroutineScope.numbersFrom(start: Int) = produce {
    var x = start
    while (true) send(x++)
}
```

以下管道过滤传入的数字流，删除所有可被整除的数字：

```kotlin
fun CoroutineScope.filter(numbers: ReceiveChannel<Int>, prime: Int) = produce {
    for (x in numbers) if (x % prime != 0) send(x)
}
```

现在我们通过从 2 开始一个数字流来构建我们的管道，从当前通道中获取一个质数，并为找到的质数启动新的管道阶段：

```kotlin
numbersFrom(2) -> filter(2) -> filter(3) -> filter(5) -> filter(7) ...
```

下面的示例打印前十个素数，在主线程的上下文中运行整个管道。在打印前十个素数后，我们使用 `cancelChildren` 扩展函数取消所有子协程。

```kotlin
var cur = numbersFrom(2)
repeat(10) {
    val prime = cur.receive()
    println(prime)
    cur = filter(cur, prime)
}
coroutineContext.cancelChildren() 
```

##### 扇出

多个协程可以从同一个通道接收，在它们之间分配工作。

```kotlin
fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) {
        send(x++)
        delay(100)
    }
}
```

然后我们可以有几个处理协程。在这个例子中，他们只打印他们的 id 和收到的号码：

```kotlin
fun CoroutineScope.launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
    for (msg in channel) {
        println("Processor #$id received $msg")
    }
}
```

现在让我们启动五个协程处理器，让他们工作近1秒：

```kotlin
val producer = produceNumbers()
repeat(5) { launchProcessor(it, producer) }
delay(9500)
producer.cancel()
```

请注意如何使用 `for` 循环显式迭代 channel 以在 `launchProcessor` 代码中执行扇出。与 `consumeEach` 不同，这个 `for` 循环模式在多个协程中使用是完全安全的。如果其中一个处理器协程失败，那么其他协程仍将处理该通道，而通过 `consumerEach` 写入的处理器总是在其正常或异常完成时消耗（取消）底层通道。

#### 扇入

多个协程可以发送到同一个通道。例如，有一个字符串通道挂起函数，它以指定的延迟将指定的字符串重复发送到该通道：

```kotlin
suspend fun sendString(channel: SendChannel<String>, s: String, time: Long) {
    while (true) {
        delay(time)
        channel.send(s)
    }
}
```

现在，让我们看看如果我们启动几个发送字符串的协程会发生什么：

```kotlin
val channel = Channel<String>()
launch { sendString(channel, "foo", 200L) }
launch { sendString(channel, "BAR", 500L) }
repeat(6) {
    println(channel.receive())
}
coroutineContext.cancelChildren()

>>foo
>>foo
>>BAR!
>>foo
>>foo
>>foo
```

#### 缓冲通道

到目前为止展示的`channel`没有缓冲区。当发送方和接收方相遇时，无缓冲通道传输数据。如果先调用 `send`，则挂起直到调用 `receive`，如果先调用 `receive`，则挂起直到调用 `send`。

所有的 `Channel()` 工厂函数和 `produce` 构建器都有可选参数来指定缓冲区大小。`Buffer` 允许发送者在挂起之前发送多个元素，类似于具有指定容量的 `BlockingQueue`，它在缓冲区已满时阻塞。

看看以下代码的行为：

```kotlin
val channel = Channel<Int>(4)
val sender = launch {
    repeat(10) {
        println("Sending $it")
        channel.send(it)
    }
}
delay(1000)
sender.cancel()

>>Sending 0
>>Sending 1
>>Sending 2
>>Sending 3
>>Sending 4
```

它使用容量为四的缓冲通道打印五次发送。

前四个元素被添加到缓冲区，发送方尝试发送第五个元素时挂起。

#### 渠道是公平的

对通道的发送和接收操作在多个协程的调用顺序方便时公平的。它们以先进先出的顺序提供。在以下示例中，`ping`和`pong`正在从共享的`table`通道接收`ball`对象。

```kotlin
data class Ball(var hits: Int)

fun main() = runBlocking {
    val table = Channel<Ball>()
    launch { player("ping", table) }
    launch { player("pong", table) }
    table.send(Ball(0))
    delay(1000)
    coroutineContext.cancelChildren()
}

suspend fun player(name: String, table: Channel<Ball>) {
    for (ball in table) {
        ball.hits++
        println("$name $ball")
        delay(300)
        table.send(ball)
    }
}

>>ping Ball(hits=1)
>>pong Ball(hits=2)
>>ping Ball(hits=3)
>>pong Ball(hits=4)
```

`ping` 协程首先启动，因为它是第一个收到球的。即使 `ping` 协程在将球发送回桌子后立即再次开始接收球，球还是被 `pong` 协程接收，因为它已经在等待它。

#### Ticker channels

`Ticker channel` 是一个特殊的集合频道，自该频道上次消费以来，每次给定延迟时间都会产生`Unit`。虽然它看起来是无用的，但它是创建复杂的基于时间的生产管道和操作符的有用构建块，可以进行窗口化和其他时间相关的处理。可以选择使用`Ticker`来执行`on tick` 操作。

要创建这样的通道，请使用工厂方法代码。要指示不需要其它元素，请在其上使用`ReceiveChannel.cannel`方法。

现在让我们看看它在实践中是如何工作的：

```kotlin
val tickerChannel = ticker(delayMillis = 100, initialDelayMillis = 0) // create ticker channel
var nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
println("Initial element is available immediately: $nextElement") // no initial delay

nextElement = withTimeoutOrNull(50) { tickerChannel.receive() } // all subsequent elements have 100ms delay
println("Next element is not ready in 50 ms: $nextElement")

nextElement = withTimeoutOrNull(60) { tickerChannel.receive() }
println("Next element is ready in 100 ms: $nextElement")

// Emulate large consumption delays
println("Consumer pauses for 150ms")
delay(150)
// Next element is available immediately
nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
println("Next element is available immediately after large consumer delay: $nextElement")
// Note that the pause between `receive` calls is taken into account and next element arrives faster
nextElement = withTimeoutOrNull(60) { tickerChannel.receive() }
println("Next element is ready in 50ms after consumer pause in 150ms: $nextElement")

tickerChannel.cancel() // indicate that no more elements are needed
```

请注意，`ticker`直到可能的消费者暂停，并且默认情况下，如果发生暂停，则调整下一个生产元素的延迟，尝试保持生产元素的固定速率。

或者，可以指定`TickerMode.FIXED_DELAY` 的模式参数以保持元素之间的固定延迟。
