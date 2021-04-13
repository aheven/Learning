### customView 组件

主要包含了`ViewDragHelper`类，一般会自动导入到工程。

ViewDragHelper是针对 ViewGroup 中的拖拽和重新定位 views 操作时提供了一系列非常有用的方法和状态追踪。基本上使用在自定义ViewGroup处理拖拽中！

#### 依赖声明

```groovy
dependencies {
    implementation "androidx.customview:customview:1.1.0"
}
```

#### 基本使用

##### 1.创建Callbak

```kotlin
class MyDragHelper : ViewDragHelper.Callback() {
    private var mLeft = 0
    private var mTop = 0

    var onViewReleasedListener: ((Int, Int) -> Unit)? = null

    override fun tryCaptureView(child: View, pointerId: Int): Boolean {
        //注意这里一定要返回true，否则后续的拖拽回调是不会生效的
        return true
    }

    //水平拖拽的时候回调的方法
    override fun clampViewPositionHorizontal(child: View, left: Int, dx: Int): Int {
        return left
    }

    override fun clampViewPositionVertical(child: View, top: Int, dy: Int): Int {
        return top
    }

    //当子View可消费点击事件的时候，重写方法使View可移动，shouldInterceptTouchEvent返回true
    override fun getViewVerticalDragRange(child: View): Int {
        return child.measuredHeight
    }

    override fun getViewHorizontalDragRange(child: View): Int {
        return child.measuredWidth
    }

    override fun onViewCaptured(capturedChild: View, activePointerId: Int) {
        super.onViewCaptured(capturedChild, activePointerId)
        mLeft = capturedChild.left
        mTop = capturedChild.top
    }

    override fun onViewReleased(releasedChild: View, xvel: Float, yvel: Float) {
        super.onViewReleased(releasedChild, xvel, yvel)
        onViewReleasedListener?.invoke(mLeft, mTop)
    }
}
```

##### 2.初始化并关联事件

```kotlin
class MyDragLinearLayout @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet?,
    defStyleAttr: Int = 0
) : LinearLayoutCompat(context, attrs, defStyleAttr) {
    private val myDragHelper: MyDragHelper = MyDragHelper()
    private val mViewDragHelper = ViewDragHelper.create(this, myDragHelper)

    init {
        myDragHelper.onViewReleasedListener = { left, top ->
            mViewDragHelper.settleCapturedViewAt(left, top)
            invalidate()
        }
    }

    override fun onInterceptTouchEvent(ev: MotionEvent): Boolean {
        return mViewDragHelper.shouldInterceptTouchEvent(ev)
    }

    override fun onTouchEvent(event: MotionEvent): Boolean {
        mViewDragHelper.processTouchEvent(event)
        return true
    }

    override fun computeScroll() {
        if (mViewDragHelper.continueSettling(true)) {
            invalidate()
        }
    }
}
```

#### 版本说明

##### 版本1.1.0

- 为布局添加了新的`Openable`接口，可以在“打开”和“关闭”状态之间转换。