### CoordinatorLayout 组件

定义顶级应用控件，控制子View的活动状态。例如 AppBarLayout 和 FloatingActionButton。

#### 依赖声明

```groovy
dependencies {
    implementation "androidx.coordinatorlayout:coordinatorlayout:1.1.0"
}
```

#### 版本说明

##### 版本 1.1.0

- CoordinatorLayout 现在实现了`NestedScrollingParent3`，并且`CoordinatorLayout.Behavior`实现了`onNestedScroll`的新重载，以使`Behaviors`能够向嵌套滚动子级报告其在每一遍`dispatchNestedScroll()`/`onNestedScroll()`操作期间所消耗的滚动距离。弃用了之前存在的`onNestedScroll(CoordinatorLayout,V,View,int,int,int,int,int)`，转为使用新的`onNestedScroll(CoordinatorLayout,V,View,int,int,int,int,int,int[])`，并且应相应地更新`Behavior`实现。
- 向无障碍服务公开了 CoordinatorLayout
- 已弃用`CoordinatorLayout.DefaultBehavior`注释。请改用`CoordinatorLayout.AttachedBehavior`接口。

#### 自定义 Behavior

```kotlin
/**
 * 自定义Behavior  FAB 上滑显示 下滑隐藏
 */

public class FatScrollAwareFABBehavior extends FloatingActionButton.Behavior {

    public FatScrollAwareFABBehavior(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onStartNestedScroll(@NonNull CoordinatorLayout coordinatorLayout, @NonNull FloatingActionButton child, @NonNull View directTargetChild, @NonNull View target, int axes, int type) {
        return axes == ViewCompat.SCROLL_AXIS_VERTICAL || super.onStartNestedScroll(coordinatorLayout, child, directTargetChild, target, axes, type);
    }

    @Override
    public void onNestedScroll(@NonNull CoordinatorLayout coordinatorLayout, @NonNull FloatingActionButton child, @NonNull View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, int type, @NonNull int[] consumed) {
        super.onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, type, consumed);
        if (dyConsumed > 0 && child.getVisibility() == View.VISIBLE) {
            child.setVisibility(View.INVISIBLE);
        } else if (dyConsumed < 0 && child.getVisibility() != View.VISIBLE) {
            child.show();
        }
    }
}
```

