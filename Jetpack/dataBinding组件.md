### DataBinding 组件

#### 视图绑定

通过视图绑定功能，您可以更轻松地编写可与视图交互的代码。在模块中启用视图绑定之后，系统会为该模块中的每个 XML 布局文件生成一个绑定类。绑定类的实例包含对在相应布局中具有 ID 的所有视图的直接引用。

大多数情况下，视图绑定会替代`findViewById`，同时，`kotlin-android-extensions`被废弃，Google 推荐使用 ViewBinding 替代。

##### 添加依赖

```groovy
android {
    ...
    buildFeatures{
        viewBinding true
    }
}
```

如果您希望在生成绑定类时忽略某个布局文件，请将`tools:viewBindingIgone="true"`属性添加到相应布局文件的根视图中：

```xml
<LinearLayout
        ...
        tools:viewBindingIgnore="true" >
    ...
</LinearLayout>
```

##### 在 Activity 中使用视图绑定

如需设置绑定类的实例以供 Activity 使用，请在 Activity 的`onCreate()`方法中执行以下步骤：

1. 调用生成的绑定类中包含静态`inflate()`方法。此操作会创建该绑定类的实例以供 Activity 使用。
2. 通过调用`getRoot()`方法或使用 Kotlin 属性语法获取对根视图的引用。
3. 将根视图传递到`setContentView()`，使其成为屏幕上的活动视图。

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    binding.button.setOnClickListener {
        Toast.makeText(this, "click", Toast.LENGTH_SHORT).show()
    }
}
```

##### 在 Fragment 中使用视图绑定

```kotlin
private var _binding: FragmentBlankBinding? = null
private val binding get() = _binding!!

override fun onCreateView(
    inflater: LayoutInflater, container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    _binding = FragmentBlankBinding.inflate(inflater, container, false)
    return binding.root
}

override fun onDestroy() {
    super.onDestroy()
    _binding = null
}
```

注意，Fragment的存在时间比其视图长。请务必在 Fragment 的 onDestoryView()方法中清除对绑定类实例的所有引用。

##### 与 findViewById 的区别

与使用`findViewById`相比，视图绑定具有一些很显著的优点：

- **Null 安全**：由于视图绑定会创建对视图的直接引用，因此不存在因视图 ID 无效而引发 Null  指针异常的风险。此外，如果视图仅出现在布局的某些配置中，则绑定类中包含其引用的字段会使用`@Nullable`标记。
- **类型安全**：每个绑定类中的字段均具有与它们在 XML 文件中引用的视图相匹配的类型，这意味着不存在发生类转换异常的风险。

这些差异意味着布局和代码之间的不兼容将会导致构建在编译时（而非运行时）失败。

##### 与数据绑定的对比

视图绑定和数据绑定均会生成可用于直接引用视图的绑定类。但是，视图绑定旨在处理更简单的用例，与数据绑定相比，具有以下优势：

- **更快的编译速度**：视图绑定不需要处理注释，因此编译时间更短。
- **易于使用**：视图绑定不需要特别标记的 XML 布局文件，因此在应用中采用速度更快。在模块中启用视图绑定后，它会自动应用于该模块的所有布局。

反之，与数据绑定相比，视图绑定也有以下限制：

- 视图绑定不支持布局变量或布局表达式，因此不能用于直接在 XML 布局文件中声明动态界面内容。
- 视图绑定不支持双向数据绑定。

考虑到这些因素，在某些情况下，最好在项目中同时使用视图绑定和数据绑定。可以在需要高级功能的布局中使用数据绑定，而在不需要高级功能的布局中使用视图绑定。

#### 数据绑定

使用声明性格式将布局中的界面组件绑定到应用中的数据源。

数据绑定与 Android Gradle 插件捆绑在一起。无需声明对此库的依赖项，但必须启用它。

如需启用数据绑定，请在模块的`build.gradle`文件中将`dataBinding`构建选项设置为`true`，如下所示：

```groovy
android {
    ...
    buildFeatures {
        dataBinding true
    }
}
```

##### Android Studio 对数据绑定的支持

Android Studio 支持许多用于修改数据绑定代码的功能。例如，它支持用于数据绑定表达式的以下功能：

- 语法突出显示
- 标记表达式语言语法错误
- XML 代码完成
- 包括导航和快速文档在内的引用。

**Layout Editor** 中的 Preview 窗格显示数据绑定表达式的默认值。例如，Preview 窗格会在以下示例声明的 `TextView` 控件上显示`my_default`值：

```xml
<TextView
    android:id="@+id/text_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:text="@{user.name,default=@string/hello_blank_fragment}" />
```

如果只需要在项目的设计阶段显示默认值，则可以使用`tools`属性，而不是默认表达式值。

##### 布局与绑定表达式

借助表达式语言，您可以编写表达式来处理视图分派的事件。数据绑定库会自动生成将布局中的视图与您的数据对象绑定所需的类。

数据绑定布局文件与视图绑定布局略有不同，以根标记`layout`开头，后跟`data`元素和`view`根元素。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="user"
            type="heven.holt.jetpack.bean.User" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context=".fragment.BlankFragment">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.firstName}"
            tools:text="firstName" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.lastName}"
            tools:text="lastName" />

    </LinearLayout>

</layout>
```

##### 表达式语言

###### 常见功能

表达式语言与托管代码中的表达式非常相似。您可以在表达式语言中使用以下运算符和关键字：

- 算数运算符`+ - * %`
- 字符串连接运算符`+`
- 逻辑运算符`&& ||`
- 二元运算符`& | ^`
- 一元运算符`+ - ! ~`
- 位移运算符`>> >>> <<`
- 比较运算符`== > < >= <=`（请注意，`<`需要转义为`&lt;`）
- `instanceof`
- 分组运算符`()`
- 字面量运算符 - 字符、字符串、数字、`null`
- 类型转换
- 方法调用
- 字段访问
- 数组访问`[]`
- 三元运算符`?:`

###### Null 合并运算符

如果左边运算数不是`null`，则 Null 合并运算符（`??`）选择左边运算数，如果左边运算数为`null`，则选择右边运算数。

```xml
android:text="@{user.displayName ?? user.lastName}"
```

等价于

```xml
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

###### 属性引用

表达式以使用以下格式在类中引用属性，这对于字段、getter 和 `ObservableField` 对象都一样：

```xml
android:text="@{user.lastName}"
```

###### 避免出现 Null 指针异常

生成的数据绑定代码会自动检查有没有`null`值并避免出现 Null 指针异常。例如，在表达式`@{user.name}`中，如果`user`为 Null，则为`user.name`分配默认值`null`。如果您引用`user.age`，其中 age 的类型为 `int`，则数据绑定使用默认值`0`。

###### 视图引用

表达式可以通过以下语法按 ID 引用布局中的其他视图：

```xml
<EditText
    android:id="@+id/example_text"
    android:layout_height="wrap_content"
    android:layout_width="match_parent"/>
<TextView
    android:id="@+id/example_output"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{exampleText.text}"/>
```

###### 集合

为方便起见，可用`[]`运算符访问常见集合，例如数组，列表，Map等。

```xml
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String>"/>
    <variable name="sparse" type="SparseArray&lt;String>"/>
    <variable name="map" type="Map&lt;String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"

```

您还可以使用 `object.key` 表示法在映射中引用值。例如，以上示例中的 `@{map[key]}` 可替换为 `@{map.key}`。

###### 字符串字面量

您可以使用单引号括住特性值，这样就可以在表达式中使用双引号，如下所示

```xml
android:text='@{map["firstName"]}'
```

也可以使用双引号括住特性值。如果这样做，则还应使用反单引号```将字符串量括起来：

```xml
android:text="@{map[`firstName`]}"
```

######  资源

表达式可以使用以下语法引用应用资源：

```xml
android:padding="@{large?@dimen/largePadding:@dimen/smallPadding}"
```

您可以通过提供参数来评估格式字符串和复数形式：

```xml
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```

您可以将属性引用和视图引用作为资源参数进行传递：

```xml
android:text="@{@string/example_resource(user.lastName, exampleText.text)}"
```

当一个复数带有多个参数时，您必须传递所有参数：

```xml
Have an orange
Have %d oranges

android:text="@{@plurals/orange(orangeCount, orangeCount)}"
```

某些资源需要显式类型求值，如下表所示：

| 类型              | 常规引用  | 表达式引用         |
| :---------------- | :-------- | :----------------- |
| String[]          | @array    | @stringArray       |
| int[]             | @array    | @intArray          |
| TypedArray        | @array    | @typedArray        |
| Animator          | @animator | @animator          |
| StateListAnimator | @animator | @stateListAnimator |
| color int         | @color    | @color             |
| ColorStateList    | @color    | @colorStateList    |

##### 事件处理

事件特性名称由监听器方法的名称确定，但有一些例外情况。例如，`View.OnClickListener`有一个`onClick()`方法，所以该事件的特性为`android:onClick`。

有一些专门针对点击事件的事件处理脚本，这些处理脚本需要使用除`android:onClick`以外的特性来避免冲突。您可以使用以下属性来避免这些类型的冲突：

| 类           | 监听器 setter                                   | 属性                  |
| ------------ | ----------------------------------------------- | --------------------- |
| SearchView   | setOnSearchClickListener(View.OnClickListener)  | android:onSearchClick |
| ZoomControls | setOnZoomInClickListener(View.OnClickListener)  | android:onZoomIn      |
| ZoomControls | setOnZoomOutClickListener(View.OnClickListener) | android:onZoomOut     |

您可以使用以下机制处理事件：

- 方法引用：在表达式中，您可以引用符合监听器方法签名的方法。当表达式求值结果为方法引用时，数据绑定会将对方引用和所有者对象封装到监听器中，并在目标视图上设置该监听器。如果表达式的求值结果为`null`，则数据绑定不会创建监听，而是设置`null`监听器。

  ```kotlin
  inner class MyHandlers {
      fun onClickFriends(view: View) {
          Toast.makeText(view.context, "onClickFriends", Toast.LENGTH_SHORT).show()
      }
  }
  
  binding.handlers = MyHandlers()
  ```

  ```xml
  <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:onClick="@{handlers::onClickFriends}"
      android:text='@{user.firstName ?? "china"}'
      tools:text="firstName" />
  ```

- 监听器绑定

  监听器绑定是在事件发生时运行的绑定表达式。它们类似于方法引用，但允许任意数据绑定表达式。此功能适用于 Gradle 2.0版及更高版本的 Android Gradle 插件。

  在方法引用中，方法的参数必须与事件监听器的参数匹配。

  ```kotlin
  class Presenter {
      fun onSaveClick(task: Task){}
  }
  
  android:onClick="@{() -> presenter.onSaveClick(task)}"
  android:onClick="@{(view) -> presenter.onSaveClick(task)}"
  android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
  ```

  可以使用 lambda 表达式表示多个参数：

  ```kotlin
  class Presenter {
      fun onCompletedChanged(task: Task, completed: Boolean){}
  }
  
  <CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
  ```

  如果您监听的事件返回类型不是 `void` 的值，则您的表达式也必须返回相同类型的值。例如，如果要监听长按事件，表达式应返回一个布尔值。

  ```kotlin
  //必须返回Boolean，否则编译错误
  fun onLongClick(view: View, message: String): Boolean {
      Toast.makeText(view.context, message, Toast.LENGTH_SHORT).show()
      return true
  }
  
  android:onLongClick="@{(theView) -> handlers.onLongClick(theView, `长按事件`)}"
  ```

  如果您需要将表达式与谓词（例如，三元运算符）结合使用，则可以使用 `void` 作为符号。

  ```xml
  android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
  ```

  避免使用复杂的监听器

  监听器表达式功能非常强大，可以使代码非常易于阅读。另一方面，包含复杂表达式的监听器会使布局难以阅读和维护。这些表达式应该像将可用数据从界面传递到回调方法一样简单。您应该在从监听器表达式调用的回调方法中实现业务逻辑。

  ##### 导入、变量和包含

  数据绑定提供了诸如导入、变量和包含等功能。通过导入功能，您可以轻松地在布局文件中引用类。通过变量功能，您可以描述可在绑定表达式中使用的属性。通过包含功能，您可以在整个应用中重复使用复杂的布局。

  ###### 导入

  通过导入功能，您可以轻松地在布局文件中引用类。您可以在 `data` 元素使用多个 `import` 元素，也可以不使用。以下代码示例将 `View` 类导入到布局文件中：

  ```xml
  <data>
      <import type="android.view.View"/>
  </data>
  <TextView
     android:text="@{user.lastName}"
     android:layout_width="wrap_content"
     android:layout_height="wrap_content"
     android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
  ```

  类型别名

  当类名有冲突时，其中一个类可使用别名重命名。以下示例将`com.example.real.estate` 软件包中的 `View` 类重命名为 `Vista`：

  ```xml
  <import type="android.view.View"/>
  <import type="com.example.real.estate.View"
          alias="Vista"/>
  
  ```

  导入其他类

  导入的类型可用作变量和表达式中的类型引用。

  ```xml
  <import type="heven.holt.jetpack.bean.User" />
  
  <variable
      name="user"
      type="User" />
  ```

  还可以使用导入类型来进行强转。

  ```xml
  <TextView
     android:text="@{((User)(user.connection)).lastName}"
     android:layout_width="wrap_content"
     android:layout_height="wrap_content"/>
  ```

  在表达式中引入静态字段和方法使，也可以使用导入类型。

  ```xml
  <data>
      <import type="com.example.MyStringUtils"/>
      <variable name="user" type="com.example.User"/>
  </data>
  …
  <TextView
     android:text="@{MyStringUtils.capitalize(user.lastName)}"
     android:layout_width="wrap_content"
     android:layout_height="wrap_content"/>
  ```

  ###### 变量

  您可以在`data`元素中使用多个`variable`元素。每个`variable`元素都描述了一个可以在布局上设置、并将在布局文件绑定表达式中使用的属性。

  ```xml
  <data>
      <import type="android.graphics.drawable.Drawable"/>
      <variable name="user" type="com.example.User"/>
      <variable name="image" type="Drawable"/>
      <variable name="note" type="String"/>
  </data>
  ```

  变量类型在编译时进行检查，如果变量实现`Observable`或者可观察集合，则应反映在类型中。如果该变量不实现`Observable`接口的基类或接口，则变量时“不可观察”的。

  系统会根据需要生成名为 `context` 的特殊变量，用于绑定表达式。`context` 的值是根视图的 `getContext()` 方法中的 `Context` 对象。`context` 变量会被具有该名称的显式变量声明替换。

  ###### 包含

  通过使用应用命名空间和特性中的变量名称，变量可以从包含的布局传递到被包含布局的绑定。以下示例展示了来自 `name.xml` 和 `contact.xml` 布局文件的被包含 `user` 变量：

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <layout xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:bind="http://schemas.android.com/apk/res-auto">
     <data>
         <variable name="user" type="com.example.User"/>
     </data>
     <LinearLayout
         android:orientation="vertical"
         android:layout_width="match_parent"
         android:layout_height="match_parent">
         <include layout="@layout/name"
             bind:user="@{user}"/>
         <include layout="@layout/contact"
             bind:user="@{user}"/>
     </LinearLayout>
  </layout>
  ```

  **数据绑定不支持 include 作为 merge 元素的直接子元素。**

  #### 使用可观察的数据对象

  可观察性是指一个对象将其数据变化告知其他对象的能力。通过数据绑定库，可以将对象、字段或集合变为可观察。

  ##### 可观察字段

  在创建实现`Observable`接口的类时要完成一些操作，但如果您的类只有少数几个属性，这样操作的意义不大。在这种情况下，可以使用通用`Observable`类和以下特定的类，将字段设为可观察字段：

  - `ObservableBoolean`
  - `ObservableByte`
  - `ObservableChar`
  - `ObservableShort`
  - `ObservableInt`
  - `ObservableLong`
  - `ObservableFloat`
  - `ObservableDouble`
  - `ObservableParcelable`

  如需使用可观察字段，请采用 Java 编程语言创建`public final`属性，或者在 kotlin 中创建只读属性，如下所示：

  ```kotlin
  class User {
      val firstName = ObservableField<String>()
      val lastName = ObservableField<String>()
      val age = ObservableInt()
  }
  ```

  如需访问字段值，请使用`set()`和`get()`访问器方法：

  ```kotlin
  private val user = User().apply {
      firstName.set("Heven")
      lastName.set("Holt")
      age.set(30)
  }
  ```

  ##### 可观察集合

  某些应用使用动态结构来保存数据。可观察集合允许使用键访问这些结构。当键为引用类型时，可以使用`ObservableArrayMap`类：

  ```kotlin
  private val user = ObservableArrayMap<String, Any>().apply {
      put("name", "HevenHolt")
      put("age", 30)
  }
  
  fun changeUser(view: View) {
      user["name"] = "Jeck Chen"
  }
  
  <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:onClick="@{handlers::changeUser}"
      android:text="@{user.name +`,age:`+user.age}"
      tools:text="firstName" />
  ```

  类似的还有`ObservableArrayList`。

  ##### 可观察对象

  实现`Observable`接口的类允许注册监听器，以便它们接收有关可观察对象的属性更改的通知。

  `Observable`接口具有添加和移除监听器的机制，但何时发送通知必须由您决定。为便于开发，数据绑定库提供了用于实现监听器注册机制的`BaseObservable`类。实现`BaseObservable`的数据类负责在属性更改时发出通知。具体操作过程是向 getter 分配`Bindable`注释，然后在 setter 中调用`notifyPropertyChanged()`方法。

  ```kotlin
  class User : BaseObservable() {
      @get:Bindable
      var firstName: String = ""
          set(value) {
              field = value
              notifyPropertyChanged(BR.firstName)
          }
  
      @get:Bindable
      var lastName: String = ""
          set(value) {
              field = value
              notifyPropertyChanged(BR.lastName)
          }
  }
  ```

  数据绑定在模块包中生成一个名为`BR`的类，该类包含用于数据绑定的资源 ID。在编译期间，`Bindable`注释会在`BR`类文件中生成一个条目。如果数据类的基类无法更改，`Observable`接口可以使用`PropertyChangeRegistry`对象实现，以便有效地注册和通知监听器。

  ```kotlin
  class User : Observable {
      private val propertyChangeRegistry = PropertyChangeRegistry()
  
      @get:Bindable
      var firstName: String = ""
          set(value) {
              field = value
              propertyChangeRegistry.notifyChange(this, BR.firstName)
          }
  
      @get:Bindable
      var lastName: String = ""
          set(value) {
              field = value
              propertyChangeRegistry.notifyChange(this, BR.lastName)
          }
  
      override fun removeOnPropertyChangedCallback(callback: Observable.OnPropertyChangedCallback?) {
          Log.e("remove", "callback: $callback")
          propertyChangeRegistry.remove(callback)
      }
  
      override fun addOnPropertyChangedCallback(callback: Observable.OnPropertyChangedCallback?) {
          Log.e("add", "callback: $callback")
          propertyChangeRegistry.add(callback)
      }
  }
  ```

#### 生成绑定类

生成的绑定类将布局变量与布局中的视图关联起来。绑定类的名称和包可以自定义。所有生成的绑定类都是从`ViewDataBinding`类继承而来的。

##### 创建绑定对象

将对象绑定到布局最常用的方法是绑定类上使用静态方法。

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: MyLayoutBinding = MyLayoutBinding.inflate(layoutInflater)

    setContentView(binding.root)
}
```

`inflate()`方法还有另外一个版本，这个版本不仅使用`LayoutInflater`对象，还使用`ViewGroup`对象：

```kotlin
val binding: MyLayoutBinding = MyLayoutBinding.inflate(getLayoutInflater(), viewGroup, false)
```

如果布局是使用其它机制扩充的，可单独绑定，如下所示：

```kotlin
val binding: MyLayoutBinding = MyLayoutBinding.bind(viewRoot)
```

有时，系统无法预先知道绑定类型。在这种情况下，可以使用`DataBindingUtil`类创建绑定：

```kotlin
val viewRoot = LayoutInflater.from(this).inflate(layoutId,parent,attachToParent)
val binding:ViewDataBinding? = DataBindingUtil.bind(viewRoot)
```

##### ViewStubs

与普通视图不同，`ViewStub`对象初始是一个不可见视图。由于`ViewStub`实际上会从视图层次结构中消失，因此绑定对象中的视图也必须消失，才能通过垃圾回收进行回收。由于视图时最终结果，因此`ViewStubProxy`对象将取代生成的绑定类中的`ViewStub`，让你能够访问`ViewStub`（如果存在），同时还能访问`ViewStub`扩充视图层次结构。

在扩展其它布局时，必须为新布局建立绑定。因此，`ViewStubProxy`必须监听`ViewStub`的`OnInflateListener`并在必要时建立绑定。

```kotlin
binding.viewStub.setOnInflateListener { stub, inflated ->
    (binding.viewStub.binding as ViewStubBinding).viewStubTv.text = "这是调用方法"
}
```

##### 即时绑定

当可变或可观察对象发生更改时，绑定会按照计划在下一帧之前发生更改。但有时必须立即执行绑定。要强制执行，请使用`executePendingBindings()`方法。

##### 后台线程

您可以在后台线程中更改数据模型，但前提是这个模型不是集合。数据绑定会在求值过程中对每个变量/字段进行本地化，以避免出现并发问题。

##### 自定义绑定类名称

默认情况下，绑定类是根据布局文件的名称生成的，以大写字母开头，移除下划线 ( _ )，将后一个字母大写，最后添加后缀 **Binding**。该类位于模块包下的 `databinding` 包中。例如，布局文件 `contact_item.xml` 会生成 `ContactItemBinding` 类。如果模块包是 `com.example.my.app`，则绑定类放在 `com.example.my.app.databinding` 包中。

通过调整 `data` 元素的 `class` 特性，绑定类可重命名或放置在不同的包中。例如，以下布局在当前模块的 `databinding` 包中生成 `ContactItem` 绑定类：

```kotlin
//自定义绑定类名
<data class="ContactItem">
    …
</data>
//您可以在类名前添加句点和前缀，从而在其他文件包中生成绑定类
<data class=".ContactItem">
    …
</data>
//使用完整软件包名称来生成绑定类
<data class="com.example.ContactItem">
    …
</data>
```

------

#### 绑定适配器

##### 设置特性值

只要绑定值发生改变，生成的绑定类就必须使用绑定表达式在视图上调用 setter 方法。您可以允许数据库自动选择方法、显示声明方法或提供选择方法的自定义逻辑来设置绑定。

###### 	自动选择方法

对于名为`example`的特性，库自动尝试查找接受兼容类型作为参数的方法`setExample(arg)。`

以`android:text="@{user.name}"`为例，库会查找接受`user.getName()`所返回类型的`setText(arg)`方法。如果`user.getName()`的返回类型为`String`，则库会查找接受`String`参数的`setText()`方法。如果表达式返回是`int`，则库会搜索接受`int`参数的`setText()`方法。表达式必须返回正确的类型。

即使不存在具有给定名称的特性，数据绑定也会起作用。然后，您可以使用数据绑定为任何 setter 创建特性。例如，支持类`DrawerLayout`没有任何特性，但有很多 setter 方法。以下布局会自动将`setScrimColor(int)` 和 `setDrawerListener(DrawerListener)` 方法分别用作 `app:scrimColor` 和 `app:drawerListener` 特性的 setter：

```xml
<android.support.v4.widget.DrawerLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:scrimColor="@{@color/scrim}"
    app:drawerListener="@{fragment.drawerListener}">
```

###### 	指定自定义方法名称

对于没有 setter 的方法。在这些情况下，某个特性可能会使用`BindingMethods`注释与 setter 关联。注释与类一起使用，可以包含多个`BindingMethod`注释，每个注释对应一个重命名的方法。

```kotlin
@BindingMethods(
    value = [
        BindingMethod(
            type = android.widget.TextView::class,
            attribute = "android:toast",
            method = "toast"
        )]
)
class ImageBindingAdapter(context: Context, attrs: AttributeSet?) :
    AppCompatTextView(context, attrs) {
    fun toast(message: String) {
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show()
    }
}

<heven.holt.jetpack.databinding.ImageBindingAdapter
    android:id="@+id/image_view"
    android:src="@mipmap/beautiful_girl"
    android:toast="@{`fdewafgr`}"
    android:background="@color/cardview_shadow_start_color"
    android:padding="20dp"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

大多情况下，无需在 Android 框架类中重命名 setter。特性已使用命名惯例实现，可自动查找匹配的方法。

###### 	提供自定义逻辑

一些属性需要自定义绑定逻辑。例如，`android:paddingLeft`特性没有关联的 setter，而是提供了`setPadding(left,top,right,bottom)`方法。使用`BindingAdapter`注释静态绑定适配器方法实现自定义特性 setter 的调用。

```kotlin
@BindingAdapter("android:paddingLeft")
fun setPaddingLeft(view: View, padding: Int) {
    view.setPadding(
        dp2px(view.context, padding),
        view.paddingTop,
        view.paddingRight,
        view.paddingBottom
    )
}
```

参数类型非常重要。第一个参数用于确定与特性关联的视图类型，第二个参数用于确定在给定特性的绑定表达式中接受的类型。

```kotlin
@BindingAdapter("imageUrl", "error")
fun loadImage(view: ImageView, url: String, error: Drawable) {
    Picasso.get().load(url).error(error).into(view)
}

//@drawable/venueError 引用应用中的资源。使用 @{} 将资源括起来可使其成为有效的绑定表达式。
<ImageView app:imageUrl="@{venue.imageUrl}" app:error="@{@drawable/venueError}" />
```

如果希望参数不是必选参数，则可以将适配器的可选 [`requireAll`](https://developer.android.com/reference/androidx/databinding/BindingAdapter#requireAll()) 标志设置为 `false`

```kotlin
@BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
fun setImageUrl(imageView: ImageView, url: String?, placeHolder: Drawable?) {
    if (url == null) {
        imageView.setImageDrawable(placeholder);
    } else {
        MyImageLoader.loadInto(imageView, url, placeholder);
    }
}
```

绑定适配器方法可以选择性在处理程序中使用旧值。同时获取旧值和新值的方法应该先为属性声明所有旧值，然后再声明新值，如以下示例所示：

```kotlin
@BindingAdapter("android:paddingLeft")
fun setPaddingLeft(view: View, oldPadding: Int, padding: Int) {
    Log.e("TAG", "setPaddingLeft: $oldPadding -> $padding")
    view.setPadding(
        dp2px(view.context, padding),
        view.paddingTop,
        view.paddingRight,
        view.paddingBottom
    )
}
```

事件处理只能使用抽象方法或抽象类，比如：

```kotlin
@BindingAdapter("android:onLayoutChange")
fun setOnLayoutChangeListener(
    view: View,
    oldValue: View.OnLayoutChangeListener?,
    newValue: View.OnLayoutChangeListener?
) {
    if (oldValue != null) {
        view.removeOnLayoutChangeListener(oldValue)
    }
    if (newValue != null) {
        view.addOnLayoutChangeListener(newValue)
    }
}
```

当监听器有多个方法时，必须将它拆分为多个监听器。例如，`View.OnAttachStateChangeListener`有两个方法：`OnViewAttachedToWindow(View)`和`onViewDetachedFromWindow(View)`。因为更改一个监听器也会影响另一个监听器，所以需要同时适用于两个属性的适配器。可以在注释中将`requireAll`设置为`false`，并指定并非必须为每个属性都分配绑定式，如下所示：

```kotlin
@BindingAdapter(
    "android:onViewDetachedFromWindow",
    "android:onViewAttachedToWindow",
    requireAll = false
)
fun setListener(
    view: View,
    detach: ViewBindingAdapter.OnViewDetachedFromWindow?,
    attach: ViewBindingAdapter.OnViewAttachedToWindow?
) {
    val newListener = if (detach == null && attach == null) {
        null
    } else {
        object : View.OnAttachStateChangeListener {
            override fun onViewDetachedFromWindow(v: View?) {
                detach?.onViewDetachedFromWindow(v)
            }

            override fun onViewAttachedToWindow(v: View?) {
                attach?.onViewAttachedToWindow(v)
            }
        }
    }
    val oldListener: View.OnAttachStateChangeListener? =
    	//此方法跟踪视图的侦听器。每个listenerResourceId一次只能跟踪一个监听器。当与BindingAdapters一起使用是，这对于add*Listener和remove*Listener方法非常有用。这保证了不会泄露监听器或视图，因此不会保留对这两者的强引用。
        ListenerUtil.trackListener(view, newListener, R.id.onAttachStateChangeListener)
    if (oldListener != null) {
        view.removeOnAttachStateChangeListener(oldListener)
    }
    if (newListener != null) {
        view.addOnAttachStateChangeListener(newListener)
    }
}
```

------



##### 对象转换

###### 自动转换对象

当绑定表达式返回`Object`时，库会选择用于设置属性的方法。`Object`会转换为所选方法的参数类型。对于使用`ObservableMap`类存储数据的应用，这种行为非常便捷，如以下示例所示：

```xml
<TextView
   android:text='@{userMap["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content" />
```

表达式中的`userMap`对象会返回一个值，该值会自动转换为用于设置`android:text`特性值的`setText(CharSequence)`方法中的参数类型。

###### 自定义转换

在某些情况下，需要在特定类型之间进行自定义转换。例如，视图的`android:background`特性需要`Drawable`，但指定的`color`值是证书。以下示例展示了某个属性需要`Drawable`，但结果提供了一个整数。

```xml
<View
   android:background="@{isError ? @color/red : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>

@BindingConversion
fun convertColorToDrawable(@ColorInt color: Int) = ColorDrawable(color)
```