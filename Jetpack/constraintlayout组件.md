### ConstraintLayout 组件

ConstraintLayout 可以使用扁平视图层次结构（无嵌套视图）创建复杂的大型布局。它与 RelativeLayout 相似，但是灵活性要高于 RelativeLayout ，并且更易于与 AndroidStudio 的布局编辑器配合使用。

[ConstraintLayout的使用指南](https://developer.android.com/training/constraint-layout)

#### 依赖声明

```groovy
dependencies {
    implementation "androidx.constraintlayout:constraintlayout:2.0.4"
}
```

#### 版本说明

##### 版本 2.1.0

ConstraintLayout 2.1.0 在 MotionLayout 中提供更丰富的功能，并新增了帮助程序（轮播界面等）

##### 版本 2.0.0

ConstraintLayout 2.0 新增了布局功能（虚拟布局等）以及用于简化视图动画的新类 MotionLayout。

#### 指南中未提及的特性

##### Group（Added in 1.1）

Group 可以用来控制一组 view 的可见性：

```xml
<androidx.constraintlayout.widget.Group
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="gone"
    app:constraint_referenced_ids="button3,button2" />
```

通过控制 group 的 hide/show 来直接控制一组 view(button3,button2) 的可见性。

##### Layer （Added in 2.0）

Layer 可以看作是它引用的 view 的边界（可以理解为包含这些 view 的一个 ViewGroup，但是 Layer 并不是 ViewGroup，Layer 并不会增加 view 的层级）。另外 Layer 支持对立面的 view 一起做动画。

```xml
<androidx.constraintlayout.helper.widget.Layer
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:constraint_referenced_ids="button13,button12,button9"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    android:background="#f00"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"/>
```

##### 自定义 Helper

为什么需要自定义 Helper？

- 保持 view 的层级不变，不像 ViewGroup 会增加 view 的层级；
- 封装一些特定的行为，方便复用；
- 一个 View 可以被多个 Helper 引用，可以很方便组合出一些复杂的效果出来。

如何自定义？

- Helper 持有 View 的引用，所以可以获取 view（getViews）然后操作 view；
- 提供了 onLayout 前后的 callback（updatePreLayout/updatePostLayout）
- Helper 继承了 view，所以 Helper 本身也是 view。

自定义 CircularRevealHelper 示例：

```kotlin
class CircularRevealHelper @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : ConstraintHelper(context, attrs, defStyleAttr) {
    //updatePostLayout会在 onLayout 之后调用
    override fun updatePostLayout(container: ConstraintLayout?) {
        super.updatePostLayout(container)
        val views = getViews(container)
        for (view in views) {
            //对一张图片作出 CircularReveal 的效果，ViewAnimationUtils提供了函数
            val anim = ViewAnimationUtils.createCircularReveal(
                view, view.width / 2, view.height / 2, 0f,
                hypot((view.height / 2).toDouble(), (view.width / 2).toDouble()).toFloat()
            )
            anim.duration = 3000
            anim.start()
        }
    }
}
```

有了 CircularRevealHelper 之后可以直接在 xml 里面使用,在CircularRevealHelper的constraint_referenced_ids里面指定需要做动画 view。

```xml
<heven.holt.jetpack.CircularRevealHelper
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:constraint_referenced_ids="imageView,imageView2"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

每个 view 不但可以接受一个 ConstraintHelper，还可以同时接受多个 ConstraintHelper。

##### Flow（VirtualLayout）

Flow 是 VirtualLayout，Flow 可以像 Chain 那样帮助快速横向/纵向布局 constraint_refrenced_ids 里面的元素。

通过 flow_wrapMode 可以指定排列方式，有三种模式：

- none：简单地把 constraint_refrenced_ids 里面的元素组成 chain，即使空间不够。
- chain：根据空间的大小和元素的大小组成一条或者多条 chain
- aligned：与 chain 类似，但是会对齐。

##### MotionLayout

降低开发人员在使用手势和组件动画的难度。	

使用方式可参照：[Android新控件MotionLayout介绍](https://blog.csdn.net/u013762572/article/details/90233221)