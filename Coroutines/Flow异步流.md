### Flow异步流

异步挂起函数返回单个值，但我们如何返回多个异步计算的值？这就是 Flows 的用武之地。

#### 表示多个值

可以使用集合在 Kotlin 中表示多个值。例如，我们有一个简单的函数，它返回一个包括三个数字的列表，然后使用 `forEach` 将它们打印出来：

```kotlin
fun simple(): List<Int> = listOf(1, 2, 3)
 
fun main() {
    simple().forEach { value -> println(value) } 
}
```

##### 序列

如果我们使用一些消耗 CPU 的阻塞代码来计算数字（每次计算需要100毫秒），那么我们可以使用序列来表示数字：

```kotlin
fun sample(): Sequence<Int> = sequence {
    for (i in 1..3) {
        Thread.sleep(100)
        yield(i)
    }
}

fun main() {
    sample().forEach { value ->
        println(value)
    }
}
```

此代码输出相同的数字，但在打印每个数字之前等待100毫秒。

##### 挂起函数

但是，上面的计算会阻塞正在运行代码的主线程。当这些值由异步代码计算时，我们可以使用挂起函数完成相同的功能：

```kotlin
suspend fun sample(): List<Int> {
    delay(1000)
    return listOf(1, 2, 3)
}

fun main() = runBlocking {
    sample().forEach { value ->
        println(value)
    }
}
```

##### Flows

使用 List 结果类型，意味着我们只能一次返回所有值。为了表示异步计算值流，我们可以使用 Flow 类型，就像我们将 Sequence 类型用于同步计算的值一样：

```kotlin
fun sample(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking {
    sample().collect { value ->
        println(value)
    }
}
```

此代码在打印每个数字之前等待100毫秒，而不会阻塞主线程。

请注意与前面示例中的 Flow 代码中的差异：

- Flow 类型的构建器函数成为流。
- `flow{...}`构建器块中的代码可以挂起。
- `sample`函数不需要使用`suspend`修饰。
- 值通过`emitted()`函数发射。
- 使用 `collect` 函数从流中收集值。

#### Flows 是冷的

流是类似于序列的冷流——流构建器中的代码在收集流之前不会运行：

```kotlin
fun sample(): Flow<Int> = flow {
    println("Flow started")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking {
    println("Calling simple function...")
    val flow = sample()
    println("Calling collect...")
    flow.collect { value -> println(value) }
    println("Calling collect again...")
    flow.collect { value -> println(value) }
}

>>Calling simple function...
>>Calling collect...
>>Flow started
>>1
>>2
>>3
>>Calling collect again...
>>Flow started
>>1
>>2
>>3
```

这就是`simple()`没有标记为挂起函数的一个关键原因。就其本身而言，`simple()`调用快速返回并且不等待任何事情。

#### Flow 取消基础

Flow 遵循协程的一般协作取消。和往常一样，当流在可取消的挂起函数（如延迟）中被挂起时，可以取消流收集。以下示例显示在`withTimeoutOrNull`块中运行时流如何在超时时被取消并且停止执行其代码：

```kotlin
fun sample(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking {
    withTimeoutOrNull(250) {
        sample().collect { value -> println(value) }
    }
    println("Donw")
}

>>Emitting 1
>>1
>>Emitting 2
>>2
>>Donw
```

#### Flow 创建

前面示例中的`flow{...}`构建器是最基本的构建器。还有其它构建器可以更轻松地声明流：

- `flowOf`构建器，用于定于发出一组固定值的流
- 可以使用`.asFlow()`扩展函数将各种集合和序列转换为流。

因此，打印流中从1到3的示例可以这样写：

```kotlin
(1..3).asFlow().collect { value -> println(value) }
```

#### 中间流操作符

流可以使用运算符进行转换，就像使用集合和序列一样。中间运算符应用于上游流并返回下游流。这些操作符也是冷流。对于运算符的调用本身不是挂起函数。它的运行很快，返回一个新的转换流。

基本操作符有很常规的名称，比如`map`和`filter`。和序列的重要区别在于这些运算符内部的代码块可以调用挂起函数。

例如，传入请求流可以使用 map 操作符映射到结果，即使执行请求是由挂起函数实现的长时间运行的操作：

```kotlin
suspend fun performRequest(request: Int): String {
    delay(1000)
    return "response $request"
}

fun main() = runBlocking {
    (1..3).asFlow()
        .map { request -> performRequest(request) }
        .collect { response -> println(response) }
}
```

##### `Transform`操作符

在流变换中，最通用的一种叫做变换运算。它可以用来进行简单的转换，比如`map`和`filter`，也可以实现更复杂的转换。使用变换运算符，我们可以任意次数地转换成任意值。

例如，使用`transform`操作符，我们可以在执行长时间运行的异步请求之前发出一个字符串，并在其后响应：

```kotlin
(1..3).asFlow()
    .transform { request ->
        emit("Making request $request")
        emit(performRequest(request))
    }.collect { response -> println(response) }

>>response 1
>>Making request 2
>>response 2
>>Making request 3
>>response 3
```

##### `take`操作符

当达到相应的限制时，像`take`这样的大小限制中间操作符会取消流程的执行。协程中的取消总是通过抛异常来执行，以便所以资源管理功能（如`try{...}finally{...}`）模块在取消的情况下正常运行：

```kotlin
fun numbers(): Flow<Int> = flow {
    try {
        emit(1)
        emit(2)
        println("This line will not execute")
        emit(3)
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking {
    numbers()
        .take(2)
        .collect { value -> println(value) }
}

>>1
>>2
>>Finally in numbers
```

这段代码的输出清楚地表明，`numbers()`函数中的`flow{...}`主体的执行在发出第二个数字后停止。

#### 终端操作符

终端操作符是暂停/启动流的函数。`collect`操作符是最基本的操作符，但还有其它的终端操作符。它们使用起来更为简单：

- 转换为各种集合，如`toList`和`toSet`。
- `first`和`single`操作符，获取第一个值并确保发出单个值的运算符。
- `reduce`将流减少到一个值。

```kotlin
val sum = (1..5).asFlow()
    .map { it * it }
    .reduce { a, b ->
        println("a=$a,b=$b")
        a + b
    }
println(sum)

>>a=1,b=4
>>a=5,b=9
>>a=14,b=16
>>a=30,b=25
>>55
```

#### Flows 是连续的

除非使用对多个流进行操作的特殊运算符，否则流的每个单独集合都按顺序执行。该集合直接在调用终端运算符的协程中工作。默认情况下不会启动新的协程。每个发出的值都由上游到下游的所有中间运营商处理，然后交付给终端的运营商。

参阅以下示例，该示例过滤偶数整数并将它们映射到字符串：

```kotlin
(1..5).asFlow()
    .filter {
        println("Filter $it")
        it % 2 == 0
    }
    .map {
        println("Map $it")
        "string $it"
    }.collect {
        println("Collect $it")
    }
```

#### Flow上下文

流的`collect`总是发生在调用协程的上下文中。例如，假如有一个`flow`协程，那么这段代码运行在指定的上下文中，而不管`flow`的实现细节：

```kotlin
withContext(context) {
    simple().collect { value ->
    	println(value)                 
    }
}
```

流的此属性称为上下文保留。

因此，默认情况下，`flow{...}`中的代码在流的收集器提供的上下文中运行。

```kotlin
fun simple(): Flow<Int> = flow {
    log("Started simple flow")
    for (i in 1..3) {
        emit(i)
    }
}

fun main() = runBlocking {
    simple().collect { value -> log("Collected $value") }
}

>>[main @coroutine#1] Started simple flow
>>[main @coroutine#1] Collected 1
>>[main @coroutine#1] Collected 2
>>[main @coroutine#1] Collected 3
```

由于`simple().collect`是从主线程调用的，`simple`的流主体也在主线程中被调用。

##### 使用`withContext`切换流上下文是错误的

但是，长时间运行 CPU 消耗代码可能需要在 Dispatchers 的上下文中执行。UI 更新代码可能需要在`Dispatchers.Main`的上下文中执行。通常，`withContext`用于切换协程的上下文，但是在`flow{...}`构建器中的代码必须遵循上下文保留属性，并且不允许从不同的上下文发出。

尝试运行这段代码：

```kotlin
fun simple(): Flow<Int> = flow {
    withContext(Dispatchers.Default) {
        for (i in 1..3) {
            emit(i)
        }
    }
}

fun main() = runBlocking {
    simple().collect { value -> println(value) }
}
```

此代码收到异常：

```kotlin
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@5511c7f8, BlockingEventLoop@2eac3323],
		but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@2dae0000, Dispatchers.Default].
		Please refer to 'flow' documentation or use 'flowOn' instead
	at ...
```

##### 正确方式是使用`flowOn`操作符

上述异常中提示使用`flowOn`函数替代。下面的示例显示了更改流上下文的正确方法：

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        log("Emitting $i")
        emit(i)
    }
}.flowOn(Dispatchers.Default)

fun main() = runBlocking {
    simple().collect { value -> log("Collected $value") }
}

>>[DefaultDispatcher-worker-1 @coroutine#2] Emitting 1
>>[DefaultDispatcher-worker-1 @coroutine#2] Emitting 2
>>[DefaultDispatcher-worker-1 @coroutine#2] Emitting 3
>>[main @coroutine#1] Collected 1
>>[main @coroutine#1] Collected 2
>>[main @coroutine#1] Collected 3
```

此时`flow{...}`在后台线程中工作，而收集发生在主线程中：

这里要注意的另一件事是`flowOn`运算符更改了流的默认顺序性质。流不再是连续的处理数据。当上游流必须在其上下文中更改`CoroutineDispatcher`时，`flowOn`运算符会为上游流创建另一个协程。

#### Flow 缓冲

从收集流所需的总时间来看，不同协程中运行流的速度可能不一样。尤其是在涉及长时间运行的异步操作时。例如，考虑一个流的发射速度为100毫秒，收集速度为300毫秒。我们来看看收集这样一个三个数字的流所需时长：

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking {
    val time = measureTimeMillis {
        simple().collect { value ->
            delay(300)
            println(value)
        }
    }
    println("Collected in $time ms")
}

>>Collected in 1220 ms
```

我们可以使用`buffer`操作符来收集数据，而不是顺序运行它们：

```kotlin
val time = measureTimeMillis {
    simple()
        .buffer()
        .collect { value ->
            delay(300)
            println(value)
        }
}
println("Collected in $time ms")

>>Collected in 1085 ms
```

只需要为第一个数字输在等待100毫秒，然后为每个数据处理等待300毫秒，这样运行大约需要1000毫秒。

请注意，`flowOn`运算符在必须更改`CoroutineDispatcher`时使用相同的缓冲机制，因此这里我们显式请求缓冲而不是更改其执行上下文。

##### 合并运算符（`conflate`）

当收集流时，可能不需要处理每个值，而只是处理最近的值。在这种情况下，当收集器处理太慢而无法处理中间值时，可以使用`conflation`操作符跳过中间值。

基于上一个示例：

```kotlin
val time = measureTimeMillis {
    simple()
        .conflate()
        .collect { value ->
            delay(300)
            println(value)
        }
}
println("Collected in $time ms")

>>1
>>3
>>Collected in 792 ms
```

可以看到，当第一个数字仍在处理中时，第二个和第三个数字已经生成，因此第二个数字被合并，只有最新的（第三个）被交付给收集器。

##### 处理最新的值（`collectLatest`）

当发射与收集都很慢时，`conflate`是一种加速处理的方式，它通过删除发出的值来实现。另一种方法是取消慢速收集器并在每次发出新值的时候再启动它。有一系列`xxxLatest`运算符执行与`xxx`运算符相同的基本逻辑，但在新值上取消其块中的代码。

让我们尝试将前面示例中的`conflate`更改为`collectLatest`：

```kotlin
val time = measureTimeMillis {
    simple()
        .collectLatest { value ->
            println("Collecting $value")
            delay(300)
            println("Done $value")
        }
}
println("Collected in $time ms")

>>Collecting 1
>>Collecting 2
>>Collecting 3
>>Done 3
>>Collected in 761 ms
```

#### 组合多个Flows

有很多方法可以组合多个流。

##### Zip

就像 Kotlin 标准库中的 `Sequence.zip` 扩展函数一样，流有一个`zip`运算符，它组合了两个流的对应值：

```kotlin
val nums = (1..3).asFlow()
val strs = flowOf("one", "two", "three")
nums.zip(strs) { a, b -> "$a -> $b" }
    .collect { println(it) }

>>1 -> one
>>2 -> two
>>3 -> three
```

##### Combine

当流在计算时，可能需要执行依赖于相应流的最新值计算，并在上游重新计算发出一个值，每次新值都将重新参与运算。这种运算符被称为组合（`Combine`）。

例如，如果前面示例中的数字每300毫秒更新一次，但字符串每400毫秒更新一次，那么使用`zip`运算符压缩它们仍然会产生相同的结果，尽管结果每400毫秒打印一次：

```kotlin
val nums = (1..3).asFlow().onEach { delay(300) }
val strs = flowOf("one", "two", "three").onEach { delay(400) }
val startTime = System.currentTimeMillis()
nums.zip(strs) { a, b -> "$a -> $b" }
    .collect { value ->
        println("$value at ${System.currentTimeMillis() - startTime} ms from start")
    }

>>1 -> one at 445 ms from start
>>2 -> two at 846 ms from start
>>3 -> three at 1248 ms from start
```

但是，在此处使用`combine`而不是`zip`的时候：

```kotlin
val nums = (1..3).asFlow().onEach { delay(300) }
val strs = flowOf("one", "two", "three").onEach { delay(400) }
val startTime = System.currentTimeMillis()
nums.combine(strs) { a, b -> "$a -> $b" }
    .collect { value ->
        println("$value at ${System.currentTimeMillis() - startTime} ms from start")
    }

>>1 -> one at 470 ms from start
>>2 -> one at 672 ms from start
>>2 -> two at 871 ms from start
>>3 -> two at 974 ms from start
>>3 -> three at 1272 ms from start
```

我们得到了完全不同的输出，其中在`nums`或`strs`流的每次发射时打印一行。

#### 扁平化Flows

Flows 表示异步接受值序列，因此很容易陷入每个值触发对另一个值序列的请求情况。例如，我们可以使用以下函数返回相距500毫秒的两个字符串流：

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i: Second")
}
```

现在，如果我们有一个由三个整数组成的流，并像这样为每个整数调用`requestFlow`：

```kotlin
(1..3).asFlow().map { requestFlow(it) }
```

然后我们最终得到一个流，需要将其展平为单个流以进行进一步处理。集合和序列为此有`flatten`和`flatMap`运算符。然而，由于流的异步特性，他们需要不同的扁平化模式，因此，在流上有一系列扁平化操作符。

##### `flatMapConcat`

连接模式由`flatMapConcat`和`flattenConcat`运算符实现。他们在开始收集下一个之前等待内部流完成，如下例所示：

```kotlin
val startTime = System.currentTimeMillis()
(1..3).asFlow().onEach { delay(100) }
    .flatMapConcat { requestFlow(it) }
    .collect { value ->
        println("$value at ${System.currentTimeMillis() - startTime} ms from start")
    }

>>1: First at 132 ms from start
>>1: Second at 633 ms from start
>>2: First at 734 ms from start
>>2: Second at 1235 ms from start
>>3: First at 1337 ms from start
>>3: Second at 1839 ms from start
```

##### `flatMapMerge`

另一种扁平化模式是同时收集所有传入的流并将它们的值合并到一个流中，以便尽快将值发出。它由`flatMapMerge`和`flattenMerge`运算符实现。它们都接受一个可选的并发参数，该参数限制同时收集的并发流数量（默认情况下等于DEFAULT_CONCURRENCY）。

```kotlin
val startTime = System.currentTimeMillis()
(1..3).asFlow().onEach { delay(100) }
    .flatMapMerge { requestFlow(it) }
    .collect { value ->
        println("$value at ${System.currentTimeMillis() - startTime} ms from start")
    }

>>1: First at 188 ms from start
>>2: First at 281 ms from start
>>3: First at 383 ms from start
>>1: Second at 689 ms from start
>>2: Second at 783 ms from start
>>3: Second at 885 ms from start
```

##### `flatMapLatest﻿`

与“处理最新值”部分中显示的`collectLatest`运算符类似，存在相应的`最新`展平模式，其中一旦发出新流，就取消先前流的集合。它由`flatMapLatest`运算符实现。

```kotlin
val startTime = System.currentTimeMillis()
(1..3).asFlow().onEach { delay(100) }
    .flatMapLatest { requestFlow(it) }
    .collect { value ->
        println("$value at ${System.currentTimeMillis() - startTime} ms from start")
    }

>>1: First at 172 ms from start
>>2: First at 350 ms from start
>>3: First at 452 ms from start
>>3: Second at 953 ms from start
```

#### Flow 异常

当操作符内的发射器或代码抛出异常时，流收集可以以异常结束。有几种方法可以处理这些异常。

##### 使用try catch 捕捉

收集器以使用`try/catch`块来处理异常：

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

fun main(): Unit = runBlocking {
    try {
        simple().collect { value ->
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Caught $e")
    }
}

>>Emitting 1
>>1
>>Emitting 2
>>2
>>Caught java.lang.IllegalStateException: Collected 2
```

##### 包括发射器也能被捕捉

前面的示例捕获了收集器中的异常，实际上，发射器或终端操作符中发生的任何异常都能被捕捉。

```kotlin
fun simple(): Flow<String> =
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
        .map { value ->
            check(value <= 1) { "Crashed on $value" }
            "string $value"
        }

fun main(): Unit = runBlocking {
    try {
        simple().collect { value ->
            println(value)
        }
    } catch (e: Throwable) {
        println("Caught $e")
    }
}
```

#### 异常透明

发射器代码如何封装它的异常处理行为呢？

发射器可以使用`catch`操作符来保持异常透明性并允许对其异常处理进行封装。catch运算符的主体可以分析异常并根据捕获的异常以不同的方式对其做出反应：

- 可以使用`throw`重新抛出异常。
- 可以使用`catch`主体中的`emit`将异常转换为值的发射。
- 可以忽略、记录或处理异常

例如，让我们在补货时发出文本：

```kotlin
simple()
    .catch { e -> emit("Caught $e") }
    .collect { value -> println(value) }
```

示例的输出是相同的，即使我们不再在代码周围使用`try/catch`。

##### 透明捕获

`catch`运算符符合异常的透明性，但是只捕获上游异常（即来自`catch`之上的所有运算符异常，但是不捕获其下）。如果`collect{...}`中的块抛出异常，则依然会抛出异常。

```kotlin
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> println("Caught $e") } // does not catch downstream exceptions
        .collect { value ->
            check(value <= 1) { "Collected $value" }
            println(value)
        }
}

>>Emitting 1
>>1
>>Emitting 2
>>Exception in thread "main" java.lang.IllegalStateException: Collected 2
	at ...
```

##### 声明式捕获

我们可以将`catch`运算符的声明性质与处理异常`collect`结合起来，通过将`collect`运算符的主体移动到`onEach`并将其放在`catch`之前。此流的收集则必须由不带参数的`collect()`调用触发：

```kotlin
simple()
    .onEach { value ->
        check(value <= 1) { "Collected $value" }
        println(value)
    }
    .catch { e -> println("Caught $e") }
    .collect()
```

现在异常不被抛出，而是打印了`Caught...`，因此我们可以在不显式使用`try/catch`块的情况下捕获所有异常。

#### Flow 完成

当流收集完成（正常或异常）时，它可能要执行一个操作。它可以通过两种方式完成：命令式或声明式。

##### 命令`finally`块

除了`try/catch`，收集器还可以使用`finally`块在收集完成时执行操作。

```kotlin
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking {
    try {
        simple().collect { value -> println(value) }
    } finally {
        println("Done")
    }
}

>>1
>>2
>>3
>>Done
```

##### 声明式处理，使用`onCompletion`操作符

可以使用`onCompletion`运算符重写前面的示例并产生相同的输出：

```kotlin
simple()
    .onCompletion { println("Done") }
    .collect { value -> println(value) }
```

`onCompletion`的主要优点是`lamabda`的一个可为空的`Throwable`参数，可用于确定流收集是正常完成还是异常完成。在以下示例中，`simple`流在发出数字1后抛出异常：

```kotlin
fun simple(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking {
    simple()
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") }
        .catch { cause -> println("Caught exception") }
        .collect { value -> println(value) }
}

>>1
>>Flow completed exceptionally
>>Caught exception
```

与`catch`不同，`onCompletion`运算符不处理异常。从上面的代码示例可以看到，异常仍然流向下游。它将被传递给更多的`onCompletion`操作符，并且可以使用`catch`操作符进行处理。

##### 顺利完成

与`catch`运算符的另一个区别是`onCompletion`会看到所有的异常，并且仅在成功完成上游流（没有取消或失败）时才会收到空异常。

```kotlin
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { cause -> println("Flow completed with $cause") }
        .collect { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}

>>1
>>Flow completed with java.lang.IllegalStateException: Collected 2
>>Exception in thread "main" java.lang.IllegalStateException: Collected 2
```

我们可以看到`cause`不为空，因为下游异常导致流中止，`onCompletion`也能观测到。

#### 命令式与声明式

现在我们知道如何收集流，并以命令式和声明式两种方式处理其完成和异常。

这里的自然问题是，哪种方法更受欢迎，为什么？

作为一个库，我们不提倡任何特定的方法，并且认为这两个选项都是有效的，应该根据您自己的喜好和代码风格进行选择。

#### 启动 flow

使用流来表示来自某个源的异步事件很容易。在这种情况下，我们需要一个类似`addEventListener`的函数来注册一段代码，对传入的事件做出反应并继续进一步工作。`onEach`操作符可以充当这个角色。然而，`onEach`是一个中间操作符。我们还需要一个终端操作员来收集流。否则，仅调用`onEach`无效。

如果我们在`onEach`之后使用`collect`终端操作符，那么它后面的代码会一直等到流被收集：

```kotlin
fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .collect()
    println("Done")
}

>>Event: 1
>>Event: 2
>>Event: 3
>>Done
```

`launchIn`操作符在这里派上用场。通过将`collect`替换为`launchIn`，我们可以在单独的协程中启动流的集合，以便立即执行其他代码：

```kotlin
events()
    .onEach { event -> println("Event: $event") }
    .launchIn(this)
println("Done")

>>Done
>>Event: 1
>>Event: 2
>>Event: 3
```

`launchIn`的参数必须指定一个`CoroutineScope`，在其中启动收集流的协程。在上面的例子中，这个作用域来自`runBlocking`协程构建器，所以当流运行时，这个`runBlocking`作用域会等待它的子协程完成，并防止主函数返回终止程序。

在实际应用中，范围将来自具有生命周期的实体。一旦该实体的生命周期终止，相应的范围就会被取消，从而取消相应流的收集。这样，`onEach{}.launch(scope)`就像`addEventListener`一样工作。但是，不需要相应的`removeEventListener`。

注意`launchIn`也返回一个Job，可以用来取消对应的流集合协程，而不取消整个范围的协程。

##### Flow 取消检查

为方便起见，流构建器对每个发出的值执行额外的`ensureActive`检查以取消。这意味从`flow{...}`发出的循环是可以立即取消的。

```kotlin
fun foo(): Flow<Int> = flow {
    for (i in 1..5) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    foo().collect { value ->
        if (value == 3) cancel()
        println(value)
    }
}

>>Emitting 1
>>1
>>Emitting 2
>>2
>>Emitting 3
>>3
>>Emitting 4
>>Exception in thread "main" kotlinx.coroutines.JobCancellationException: >>BlockingCoroutine was cancelled; 
```

但是，出于性能原因，大多数其它流操作符不会自行额外的取消检查。例如，如果您使用`IntRange.asFlow`扩展来编写相同的循环并且不在任何地方挂起，则不会取消检查：

```kotlin
fun main() = runBlocking<Unit> {
    (1..5).asFlow().collect { value ->
        if (value == 3) cancel()
        println(value)
    }
}

>>1
>>2
>>3
>>4
>>5
>>Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled;
```

##### 使繁忙的流程可取消

如果您有一个带有协程的繁忙循环，您必须明确检查取消。您可以添加`.onEach{ currentCoroutineContext().ensureActive() }`。同时，Kotlin 添加了一个随时可用的可取消的运算符来做到这一点：

```kotlin
fun main() = runBlocking<Unit> {
    (1..5).asFlow()
        .cancellable()
        .collect { value ->
            if (value == 3) cancel()
            println(value)
        }
}

>>1
>>2
>>3
>>Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled;
```

#### Flow 与 Reactive Streams

对于熟悉 Reactive Stream 或响应式框架（如 RxJava 和 Reactor）的人来说，Flow 的设计可能看起来非常熟悉。

事实上，它的设计灵感来自 Reactive Stream 及其各种实现。但是 Flow 的主要目标是尽可能简单的设计，对 Kotlin 的友好支持，并支持结构化并发。
