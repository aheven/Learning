# lifecycle组件

## 声明依赖项

```xml
dependencies {
    def lifecycle_version = "2.3.1"
    def arch_version = "2.1.0"

    // ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    // LiveData
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
    // Lifecycles only (without ViewModel or LiveData)
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version"

    // Saved state module for ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:$lifecycle_version"

    // Jetpack Compose Integration for ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-compose:1.0.0-alpha04"

    // Annotation processor
    kapt "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
    // alternately - if using Java8, use the following instead of lifecycle-compiler
    implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"

    // optional - helpers for implementing LifecycleOwner in a Service
    implementation "androidx.lifecycle:lifecycle-service:$lifecycle_version"

    // optional - ProcessLifecycleOwner provides a lifecycle for the whole application process
    implementation "androidx.lifecycle:lifecycle-process:$lifecycle_version"

    // optional - ReactiveStreams support for LiveData
    implementation "androidx.lifecycle:lifecycle-reactivestreams-ktx:$lifecycle_version"

    // optional - Test helpers for LiveData
    testImplementation "androidx.arch.core:core-testing:$arch_version"
}
```

## 版本更新

### V2.3.0

- **对非 Parcelable 类的 `SavedStateHandle` 支持**：`SavedStateHandle`现在允许您针对指定键调用`setSavedStateProvider()`以支持延迟序列化，从而提供`SavedStateProvider`，该构造函数会在`SavedStateHandle`被要求保存其状态时获得对`saveState()`的回调。
- **Lifecycle 行为强制执行**：
  - LifecycleRegistry 现在强制将 `DESTROYED` 作为终止状态。
  - `LifecycleRegistry` 现在会验证其方法是否在主线程上调用。它始终是 Activity、Fragment 等的生命周期的一项要求、非主线程上的观察对象增加会导致难以在运行时捕获崩溃。对于您自己的组件拥有的`LifecycleRegistry`对象，您可以通过使用`LifecycleRegistry.createUnsage(...)`明确停止检查，但之后您必须确保从其他线程访问`LifecycleRegistry`时，系统会适当的同步。
- **生命周期状态和事件辅助程序**：向`Lifecycle.Event`添加了`downFrom(State)`、`downTo(State)`、`upFrom(State)`、`upTo(State)`的静态辅助方法，以便在生成`Event`时提供`State`和过渡方向。添加了`getTargetState()`方法，该方法会提供 Lifecycle 在 `Event` 结束后将直接过渡到的 `State`。
- **`withStateAtLeast`**：添加了 `Lifecycle.withStateAtLeast` API，这些 API 会等待某个生命周期状态，并在状态发生变化时同步运行非挂起代码块，然后继续执行结果。这些 API 与现有的 `when*`方法不同，因为它们不允许运行暂停代码，且不使用自定义调度程序。
- **`ViewTree` API**：新的`ViewTreeLifecycleOwner.get(View)`和`ViewTreeViewModelStoreOwner.get(View)` API 允许您根据`View`实例分别检索包含的`LifecycleOwner`和`ViewModelStoreOwner`。您必须升级到 Activity `1.2.0` 和 Fragment `1.3.0` 以及 AppCompat 1.3.0 或更高笨笨，才能正确填充此 API。`findViewTreeLifecycleOwner` 和 `findViewTreeViewModelStoreOwner` Kotlin 扩展分别在 `lifecycle-runtime-ktx` 和 `lifecycle-viewmodel-ktx` 中提供。
- **`LiveData.observe()` Kotlin 扩展弃用**：现在，弃用了使用 Lambda 语法所需的 `LiveData.observe()` Kotlin 扩展，因为改用 Kotlin 1.4 后，便不再需要此扩展。

### V2.2.0

- **Lifecycle 协程集成**：新的`lifecycle-runtime-ktx`工件实现了 Lifecycle 协程与 Kotlin 协程之间的集成。此外，我们还扩展了 `lifecycle-livedata-ktx` 以便利用协程的优势。
- **弃用 `ViewModelProviders.of()`**：已弃用`ViewModelProviders.of()`。您可以将 `Fragment` 或 `FragmentActivity` 传递给新的 `ViewModelProvider(ViewModelStoreOwner)` 构造函数，以便在使用 Fragment `1.2.0` 时实现相同的功能。
- **弃用 `Lifecycle-extensions` 工件**：在上面弃用 `ViewModelProviders.of()` 后，此版本标志着弃用 `lifecycle-extensions` 中的最后一个API，因此现在该工件已完全被弃用。我们强烈建议依赖于您需要的特定的 lifecycle 工件（例如，如果您使用的是 `LifecycleService`，则依赖于`lifecycle-service`；如果您使用的是`ProcessLifecycleOwner`，则依赖于`lifecycle-process`）而不是`lifecycle-extensions`，因为将来不会有 `lifecycle-extensions` 的`2.3.0`版本。
- **Gradle 增量注释处理器**：默认情况下，Lifecycle 的注释处理器是增量注释处理器。如果您的应用是用 Java 8 语言编写的，您可以使用 `DefaultLifecycleObserver`；如果是用 Java7 语言编写的，您可以使用 `LifecycleEventObserver`。

### ViewModel-SavedState 版本 1.1.0

- 新增了 **SavedStateHandle** 类。它使您的`ViewModel`类能够访问和促成已保存状态。您可以在`ViewModel`类的构造函数以及 Fragment 默认提供的工厂中接收此对象，并且 AppCompatActivity 会自动注入 `SavedStateHandle`。
- 添加了 **AbstractSavedStateViewModelFactory**。它允许您为`ViewModel`自定义工厂，并向其提供`SavedStateHandle`访问权限。

### V2.1.0

- 添加了`LifecycleEventObserver`，用于应对需要生命周期流的情况。它是一个公共 API，而不是隐藏的`GenericLifecycleObserver`类。
- 为`LiveData.observe`方法和`Transformations.*`方法添加了ktx扩展程序。
- 添加了`Transformations.distinctUntilChanged`，它可以创建一个新的 LiveData 对象，该对象会在源`LiveData`指发生更改后发出一个值。
- 通过添加扩展属性`ViewModel.viewModelScope`在ViewModel 中添加了协程支持。

### V2.0.0

发布了 Lifecycle `2.0.0`，与`2.0.0-rc01`相比，该版本修复了 ViewModel 中的一个问题。

##### 问题修复

- 修复了错误地移除了构造函数的 ViewModel Proguard 规则
- 修复了 LifecycleObserver ProGuard 规则以仅保留实现而不保留子接口
- 修复了 ViewModel Proguard 规则以允许混淆和压缩

### V1.1.1

只有一项小的变更：`android.arch.core.util.Function`已从`arch:runtime`移至`arch:common`。这样，便可以在没有运行时依赖项的情况下使用它，例如在`paging:common`中。

`lifecycle:common`是`lifecycle:runtime`的依赖项，所以此变更不会直接影响`lifecycle:runtime`，而只会影响像 Paging 一样直接依赖于 `lifecycle:common`的模块。

### V1.1.0

#### 打包变更

现在可使用小得多的新增依赖项：

- `android.arch.lifecycle:livedata:1.1.0`
- `android.arch.lifecycle:viewmodel:1.1.0`

#### Api 变更

- 已弃用的`LifecycleActivity`和`LifecycleFragment`现**已移除**-请使用`FragmentActivity`、`AppCompatActivity`或支持`Fragment`。
- 向`ViewModelProviders`和`ViewModelStores`添加了`@NonNull`注释
- 弃用了`ViewModelProviders`构造函数-请直接使用其静态方法
- 弃用了`ViewModelProviders.DefaultFactory`-请使用`ViewModelProvider.AndroidViewModelFactory`
- 添加了静态`ViewModelProvider.AndroidViewModelFactory.getInstance(Application)`方法，用于检索适合创建`ViewModel`和`AndroidViewModel`实例的静态`Factory`。

## 使用生命周期感知型组件处理生命周期

生命周期感知型组件可执行操作来响应另一个组件（如 Activity 和 Fragment）的生命周期状态的变化。这些组件有助于编写出更有条理且更精简的代码。

一种常见的模式是在 Activity 和 Fragment 的生命周期方法中实现依赖组件的操作。但是，这种模式会导致代码条理性很差而且会扩散错误。通过使用生命周期感知型组件，您可以将依赖组件的代码从生命周期方法移入组件本身中。

`androidx.lifecycle`软件包提供了可用于构建生命周期感知型组件的类和接口。

## Lifecycle

`Lifecycle`是一个类，用于存储有关组件（如 Activity 和 Fragment）的生命周期状态的信息，并允许其它对象观察此状态。

`Lifecycle`使用两种主要枚举跟踪其关联组件的生命周期状态：

### 事件（Event）

​	从框架和`Lifecycle`类分派的生命周期事件。这些事件映射到 Activity 和 Fragment 中的回调事件。

### 状态（State）

​	由`Lifecycle`对象跟踪的组件的当前状态。

![构成 Android Activity 生命周期的状态和时间](C:\Users\Administrator\Documents\RxJava\pic\lifecycle-states.svg)

您可以将状态看作图中的节点，将事件看作这些节点之间的边。

类可以通过向其方法添加注解来监控组件的生命周期状态。然后，您可以通过调用`Lifecycle`类的`addObserver()`方法并传递观察者的实例来添加观察者，如以下示例中所示：

```kotlin
public class MyObserver implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
```

在上面的示例中，`myLifecycleOwner`对象实现了`LifecycleOwner`接口。

## LifecycleOwner

`LifecycleOwner`是单一方法接口，表示类具有`Lifecycle`。它具有一种方法（即`getLifeCycle()`），该方法必须由类实现。如果您尝试管理整个应用进程的生命周期，请参阅`ProcessLifecycleOwner`。

此接口从各个类（如`Fragment`和`AppCompatActivity`）抽象化`Lifecycle`的所有权，并允许编写与这些类搭配使用的组件。任何自定义应用类均可实现`LifecycleOwner`接口。

实现`LifecycleObserver`的组件可与实现`LifecycleOwner`的组件完美配合，因为所有者可以提供生命周期，而观察者可以注册以观察生命周期。

```kotlin
class MyObserver(private val lifecycle: Lifecycle) : LifecycleObserver {
    init {
        lifecycle.addObserver(this)
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun connectListener() {
        Log.d("MyObserver", "connectListener: onResume -> $this")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun disconnectListener() {
        Log.d("MyObserver", "disconnectListener: OnPause -> $this")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    fun removeListener() {
        lifecycle.removeObserver(this)
    }

    fun enable() {
        if (lifecycle.currentState.isAtLeast(Lifecycle.State.STARTED)) {
            ...
        }
    }
}
```

对于此实现，类可以完全感知生命周期。如果我们需要从另一个 Activity 或 Fragment 使用它，只需对其进行初始化。所有设置和拆解操作都由类本身管理。

### 实现自定义 LifecycleOwner

如果您有一个自定义类并希望使其成为 LifecycleOwner，您可以使用 LifecycleRegistry 类，但需要将事件转发到该类，如下代码示例：

```kotlin
class CustomLifecycleActivity : Activity(), LifecycleOwner {
    private val lifecycleRegistry = LifecycleRegistry(this)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        lifecycleRegistry.currentState = Lifecycle.State.CREATED
    }

    override fun onStart() {
        super.onStart()
        lifecycleRegistry.currentState - Lifecycle.State.STARTED
    }

    override fun getLifecycle(): Lifecycle = lifecycleRegistry
}
```

## 生命周期感知型组件的最佳做法

- 使界面控制器（Activity 和 Fragment）尽可能保持精简。它们不应试图获取自己的数据，而应使用`ViewModel`执行此操作，并观察`LiveData`对象以将更改体现到视图中。
- 设法编写数据驱动型界面，对于此类界面，界面控制器的责任是随着数据更改而更新试图，或者将用户操作通知给`ViewModel`。
- 将数据逻辑放在`ViewModel`类中。`ViewModel`应充当界面控制器与应用其余部分之间的连接器。不过要注意，`ViewModel`不负责获取数据（例如，从网络获取）。但是，`ViewModel`应调用相应的组件来获取数据，然后将结果提供给界面控制器。
- 使用数据绑定在视图与界面控制器之间维持干净的接口。这样一来，您可以使视图更具声明性，并尽量减少需要在 Activity 和 Fragment 中编写的更新代码。
- 如果界面很复杂，不妨考虑创建 presenter 类来处理界面的修改。这样做可使界面组件更易于测试。
- 避免在`ViewModel`中引用`View`或`Activity`上下文。如果`ViewModel`存在的时间比 Activity 更长（在配置更改的情况下），Activity 将泄露并且不会获得垃圾回收器的妥善处置。
- 使用 kotlin 协程管理长时间运行的任务和其他可以异步运行的操作。

## 生命周期感知型组件的用例

生命周期感知型组件可使您在各种情况下更轻松地管理生命周期。比如：

- 在粗粒度和细粒度位置更新之间切换。使用生命周期感知型组件可在位置应用可见时启用细粒度位置更新，并在应用切换到后台时使用粗粒度更新。借助生命周期感知型组件`LiveData`，应用可以在用户使用位置发生变化时自动更新界面。
- 停止和开始视频缓冲。使用生命周期感知型组件可尽快开始视频缓冲，但会推迟播放，直到应用完全启动。此外，应用销毁后，您还可以使用生命周期感知型组件终止缓冲。
- 开始和停止网络连接。借助生命周期感知型组件，可在应用位于前台时启用网络数据的实时更新（流式传输），并在应用进入后台时自动暂停。
- 暂停和恢复动画可绘制资源。

## 处理 ON_STOP 事件

如果`Lifecycle`属于`AppCompatActivity`或`Fragment`，那么调用`AppCompatActivity`或`Fragment`的`onSaveInstanceState()`时，`Lifecycle`的状态会更改为`CREATED`并且会分派`ON_STOP`事件。

通过`onSaveInstanceState()`保存`Fragment`或`AppCompatActivity`的状态后，其界面被视为不可变，直到调用`ON_START`。如果在保存状态后尝试修改界面，很可能会导致应用的导航状态不一致，因此应用在保存状态后运行`FragmentTransaction`时，`FragmentManager`会抛出异常。

`LiveData`本身可防止出现这种极端情况，方法是在其观察者的关联`Lifecycle`还没有至少处于`STARTED`状态时避免调用其观察者。在后台，它会在决定调用其观察者之前调用`isAtLeast()`。

遗憾的是，`AppCompatActivity`的`onStop()`方法会在`onSaveInstanceState()`之后调用，这样就会留下一个缺口，即不允许界面状态发生变化，但`Lifecycle`尚未移至`CREATED`状态。

为防止出现这个问题，`beta2`及更低版本中的`Lifecycle`类会将状态标记为`CREATED`而不分派事件，这样一来，即使未分派事件（直到系统调用`onStop()`），检查当前状态的代码也会获得实际值。

遗憾的是，此解决方案有两个主要的问题：

- 在 API 23 及更低级别，Android 系统实际上会保存 Activity 的状态，即使它的一部分被另一个 Activity 覆盖。换句话说，Android 系统会调用`onSaveInstanceState()`，但不一定会调用`onStop()`。这样可能会产生很长的时间间隔，在此时间间隔内，观察者仍认为生命周期处于活动状态，虽然无法修改其界面状态。
- 任何要向`LiveData`类公开类似行为的类都必须实现由`Lifecycle`版本`beta2`及更低版本提供的解决方案。

# LiveData 概览

`LiveData`是一种可观察的数据存储器类。与常规的可观察类不同，LiveData 具有生命周期感知能力，这种感知能力可确保 LiveData 仅更新处于活跃生命周期状态的应用组件观察者。

如果观察者（由`Observer`类表示）的生命周期处于`STARTED` 或`RESUMED`状态，则LiveData 会认为该观察者处于活跃状态。LiveData只会将更新通知给活跃的观察者。为观察`LiveData`对象而注册的非活跃观察者不会收到更改通知。

您以注册与实现`LifecycleOwner`接口的对象配对的观察者。有了这种关系，当相应的`Lifecycle`对象的状态变为`DESTROYED`时，便可移除此观察者。这对于 Activity 和 Fragment 特别有用，因为它们可以放心地观察`LiveData`对象，而不必担心内存泄漏。

## 使用LiveData 的优势

**确保界面符合数据状态**

​	LiveData 遵循观察者模式，当底层数据发生变化时，LiveData 会通知 `Observer`对象。您可以整合代码以在这些`Observer`对象中更新界面。这样一来，您无需在每次应用数据发生变化时更新界面，因为观察者会替您完成更新。

**不会发生内存泄露**

​	观察者会绑定到`Lifecycle`对象，并在其关联的生命周期遭到销毁后进行自我清理。

**不会因 Activity 停止而导致崩溃**

​	如果观察者的生命周期处于非活跃状态（如返回栈中的 Activity），则它不会接收任何 LiveData 事件。

**不再需要手动处理生命周期**

​	界面组件只是观察相关数据，不会停止或恢复观察。LiveData将自动管理所有这些操作。

**数据始终保持最新状态**

​	如果生命周期变为非活跃状态，它会在再次变为活跃状态时接收最新的数据。

**适当的配置更改**

​	如果由于配置更改（如设备旋转）而重新创建了 Activity 或 Fragment，它会立即接收最新的可用数据。

**共享资源**

​	您可以使用单例模式扩展`LiveData`对象以封装系统服务，以便在应用中共享他们。`LiveData`对象连接到系统服务一次，然后需要相应资源的任何观察者只需观察`LiveData`对象。

## 使用 LiveData 对象

1. 创建`LiveData`的实例以存储某种类型的数据。这通常在`ViewModel`类中完成。
2. 创建可定义`onChanged()`方法的`Observer`对象，该方法可以控制当`LiveData`对象存储的数据更改时会发生什么。通常情况下，您可以在界面控制器中创建`Observer`对象。
3. 使用`observe()`方法将`Observer`对象附加到`LiveData`对象。`observe()`方法会采用`LifecycleOwner`对象。这样会使`Observer`对象订阅`LiveData`对象，以使其收到有关更改的通知。通常情况下，您可以在界面控制器中附加`Observer`对象。

### 创建 LiveData 对象

LiveData 是一种可用于任何数据的封装容器，其中包括可实现`Collections`的对象，如`List`。`LiveData`对象通常存储在`ViewModel`对象中。并可通过 getter 方法进行访问，如以下示例中所示：

```kotlin
public class NameViewModel : ViewModel() {
    val currentName:MutableLiveData<String> by lazy {
        MutableLiveData<String>()
    }
}
```

注意：请确保用于更新界面的LiveData对象存储在 ViewModel 对象中，而不是将其存储在 Activity 或 Fragment 中，原因如下：

- 避免 Activity 和 Fragment 过于庞大。现在，这些界面控制器负责显示数据，但不负责存储数据状态。
- 将 LiveData 实例与特定的 Activity 或 Fragment 实例分离开，并使 LiveData 对象在配置更改后继续存在。

### 观察 LiveData 对象

在大多数情况下，应用组件的`onCreate()`方法是开始观察`LiveData`对象的正确着手点，原因如下：

- 确保系统不会从 Activity 或 Fragment 的`onResume()`方法进行冗余调用。
- 确保 Activity 或 Fragment 变为活跃状态后具有可以立即显示的数据。一旦应用组件处于`STARTED`状态，就会从它正在观察的`LiveData`对象接收最新值。只有在设置了要观察的`LiveData`对象时，才会发生这种情况。

通常，LiveData 仅在数据发生更改时才发送更新，并且仅发送给活跃观察者。此行为的一种例外情况是，观察者从非活跃状态更改为活跃状态时也会收到更新。此外，如果观察者第二次从非活跃状态更改为活跃状态，则只有在上次变为活跃状态以来值发生了更改时，它才会收到更新。

以下示例代码说明了如何开始观察`LiveData`对象：

```kotlin
class NameActivity : AppCompatActivity() {
    private val model: NameViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?){
        super.onCreate(savedInstanceState)
        
        val nameObserver = Observer<String> { newName ->
        	nameTextView.text = newName                                    
        }
        
        model.currentName.observe(this,nameObserver)
    }
}
```

在传递`nameObserver`参数的情况下调用`observe()`后，系统会立即调用`onChange()`，从而提供`mCurrentName`中存储的最新值。如果`LiveData`对象尚未在`mCurrentName`中设置值，则不会调用 `onChanged()`。

### 更新 LiveData 对象

LiveData 没有公开可用的方法来更新存储的数据。`MutableLiveData`类将公开`setValue(T)`和`postValue(T)`方法，如果需要修改存储在`LiveData`对象中的值，则必须使用这些方法。通常情况下会在`ViewModel`中使用`MutableLiveData`，然后`ViewModel`只会向观察者公开不可变的`LiveData`对象。

设置观察者关系后，您可以更新`LiveData`对象的值（如以下示例中所示）：

```kotlin
button.setOnClickListener {
    val anotherName = "John Doe"
    model.currentName.setCaue(anotherName)
}
```

注意：必须调用setValue(T)方法从主线程更新 LiveData 对象。如果在工作线程中执行代码，则可以改用postValue(T) 方法来更新LiveData 对象。

## 扩展 LiveData

```kotlin
class StockLiveData(symbol : String) : LiveData<BigDecimal>(){
    private val stockManager = StockManager(symbol)
    
    private val listener = { price:BigDecimal ->
    	value = price
    }
    
    override fun onActive() {
        stockManager.requestPriceUpdates(listener)
    }
    
    override fun onInactive() {
    	stockManager.removeUpdates(listener)
    }
}
```

本示例中的价格监听器实现包括以下重要方法：

- 当`LiveData`对象具有活跃观察者时，会调用`onActive()`方法。这意味着，您需要从此方法开始观察股价更新。
- 当`LiveData`对象没有任何活跃观察者时，会调用`onInactive()`方法。由于没有观察者在监听，因此没有理由与`StockManager`服务保持连接。
- `setValue(T)`方法将更新`LiveData`实例的值，并将更改告知活跃观察者。

## 转换 LiveData

您可能希望在将`LiveData`对象分派给观察者之前对存储在其中的值进行更改，或者可能需要根据另一个实例的值返回不同的`LiveData`实例。`Lifecycle`软件包会提供`Transformations`类，该类包括可应对这些情况的辅助方法。

- `Transformations.map()`

  对存储在`LiveData`对象中的值应用函数，并将结果传播到下游。

  ```kotlin
  val userLiveData: LiveData<User> = MutableLiveData()
  val userName: LiveData<String> =
      Transformations.map(userLiveData) { user -> "${user.firstName} ${user.lastName}" }
  ```

- `Trandformations.switchMap()`

  与`map()`类似，对存储在`LiveData`对象中的值应用函数，并将结果解封和分派到下游。传递给`switchMap()`的函数必须返回`LiveData`对象，如下示例中所示：

  ```kotlin
  private fun getUser(id: String): LiveData<User> {
    ...
  }
  val userId: LiveData<String> = ...
  val user = Transformations.switchMap(userId) { id -> getUser(id) }
  ```

您可以使用转换方法在观察者的生命周期内传送信息。除非观察者正在观察返回的`LiveData`对象，否则不会计算转换，因为转换是以延迟的方式计算，所以与生命周期相关的行为会隐式传递下去，而不需要额外的显式调用或依赖项。

如果您认为`ViewModel`对象中需要有`Lifecycle`对象，那么进行转换或许是更好的解决方案。例如，假设您有一个界面组件，该组件接受地址并返回该地址的邮政编码。您可以为此组件实现简单的`ViewModel`，如以下示例代码所示：

```kotlin
class MyViewModel(private val respository:PostalCodeRepository) : ViewModel() {
    private fun getPostalCode(address:String): LiveData<String>{
        //DON'T DO THIS
        return respository.getPostCode(address)
    }
}
```

然后，该界面组件需要取消注册先前的`LiveData`对象，并在每次调用`getPostalCode()`时注册到新的示例。此外，如果重新创建了该见面组件，它会再次触发一次对`repository.getPostCode()`方法的调用，而不是使用先前调用所得的结果。因此推荐下面的做法：

```kotlin
class MyViewModel(private val repository: PostalCodeRespository): ViewModel() {
    private val addressInput = MutableLiveData<String>()
    
    val postalCode: LiveData<String> = Trandformations.switchMap(addressInput) {
        address -> repository.getPostCode(address)
    }
    
    private fun setInput(address:String) {
        addressInput.value = address
    }
}
```

在这种情况下，`postalCode`字段定义为`addressInput`的转换。只需要您的应用具有与`postalCode`字段关联的活跃观察者，就会在`addressInput`发生更改时重新计算并检索该字段的值。

## 合并多个 LiveData 源

`MediatorLiveData`是`LiveData`的子类，允许合并多个 LiveData 源。只要任何原始的 LiveData 源对象发生更改，就会触发 `MediatorLiveData` 对象的观察者。

例如，如果界面中有可以从本地数据库或网络更新的`LiveData`对象，则可以向`MediatorLiveData`对象添加以下源：

- 与存储在数据库中的数据关联的`LiveData`对象。
- 与从网络访问的数据关联的`LiveData`对象。

您的 Activity 只需要观察 `MediatorLiveData` 对象即可从这两个源接收更新。