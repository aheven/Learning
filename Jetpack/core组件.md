### Core 组件

针对最新的平台功能和 API 调整应用，同时还支持旧设备。

下表列出了`androidx.core	`组中的所有工件：

| 工件           | 当前稳定版 | 下一候选版本 | Beta 版      | Alpha 版      |
| -------------- | ---------- | ------------ | ------------ | ------------- |
| core           | 1.3.2      | -            | 1.5.0-beta01 | -             |
| core-animation | -          | -            | -            | 1.0.0-alpha02 |
| core-role      | -          | 1.0.0-rc01   | -            | 1.1.0-alpha01 |

#### 依赖声明

```groovy
dependencies {
    def core_version = "1.3.2"

    // Java language implementation
    implementation "androidx.core:core:$core_version"
    // Kotlin
    implementation "androidx.core:core-ktx:$core_version"

    // To use RoleManagerCompat
    implementation "androidx.core:core-role:1.0.0"

    // To use the Animator APIs
    implementation "androidx.core:core-animation:1.0.0-alpha02"
    // To test the Animator APIs
    androidTestImplementation "androidx.core:core-animation-testing:1.0.0-alpha02"
}
```

#### 版本说明

##### 版本1.5.0

- 遵循关于边界兼容性/平台互操作性的 API 准则。
- 将 AppCompatEditText 中的拖放事件 与 OnReceiveContentListener 进行了集成。
- 将新的 Insets Animation API 连接到了平台实现。
- 添加了新的  Insets Animation API 。
- 更新了 OnReceiveContentListener 和相关 API：
  - 更新了 OnReceiveContentListener ，这样就可以通过 ViewCompat 在任何类型的视图中设置该方法。
  - 将参数封装到对象中的`OnReceiveContentListener.onReceiveContent()`。
  - 以参数形式向`onReceiveContent()`添加了 linkUri，以确保与键盘图片 API 的向后兼容性。
  - 向`onReceiveContent()`添加了 Bundle 参数，以确保与键盘图片 API 的向后兼容性，并且便于在将来改进 API。
  - 更新了`onReceiveContent()`，以返回未被使用的任何内容，使用委托为默认处理方法。
  - 从公共 API 中移除了`TextViewOnReceiveContentListener`，因为现在可以通过从监听器返回任何未被使用的内容。
- 已弃用`BuildCompat.isAtLeastR`。
- 已将`widget.RichContentReceiverCompat`移至`view.OnReceiveContentListener`。
- 添加了`Preconditions.checkFlagsArgument`。
- 弃用了用于出站共享的自定义菜单。
- 现在，通知可以标记为未接电话。
- 添加了`PackageInfoCompat#getSignature`，用于检索软件包的证书数组。

##### 版本1.4.0

- 添加了用于插入富媒体内容（如粘贴图片）的通用 API。新回调提供了一个 API，应用可以实现该单一 API 来支持不同方式插入富媒体内容。目前，该 API 仅添加到了`AppCompatEditText`，并将为以下代码路径调用：
  - 从剪贴板粘贴
  - 从 IME 插入内容。
- 向后移植了`androix.os.Process.isApplicationUid(int)`，有助于应用确定代码是否在独立的进程中执行。
- 向后移植了`LocusId`，有助于应用在不同子系统（例如内容捕获、快捷方式和通知）之间关联状态。
- 在 ViewGroup 添加了祖先序列。

##### 版本1.3.2

- 允许从任何 API 级别的任何生命周期状态安全地调用`ActivityCompat.recreate()`。

##### 版本1.3.0

- 新增了`NestedScrollView` API，可实现在指定时长内顺畅滚动。
- 新增了`ViewCompat` API，可用于检索已分派给视图层次结构的原始窗口

##### 版本1.2.0

- 在`NotificationCompat`中添加了新的 API 和问题修复。
- 添加了可与 Android Q 中以向后兼容方式引入`BlendMode`结合使用的新API
- 在 AccessibilityCompat 中添加了新的 API 和问题修复。
- 添加了可与`ShortcutInfo`结合使用的新 API。
- 添加了可与`WindowInsets`结合使用的新 API。
- 修复了`EditorInfoCompat`、`ShareCompat`、`WakefulBroadcastReceiver`和`InputConnectionCompat`中28.0（支持库）与1.1（AndroidX）之间的软件包秘钥字符串的向后兼容性问题。

##### 版本1.1.0

- 更新了无障碍功能 API 与 Android 10 平台的无障碍功能 API 的匹配。
- 添加了嵌套滚动改进；`NestedScrollingChild3`和`NestedScrollingParent3`。
- 此库不再在 API 中公开`androidx.collection`依赖项。需要额外依赖。
- 解决了由 androidx 重构导致的 IPC 兼容性问题
- 添加了对 AppCompat DayNight 的各种修复。

#### 各个工具类使用

1. ##### ContextCompat

```kotlin
val intent1 = Intent(this, SecondActivity::class.java)
val intent2 = Intent(this, ThridActivity::class.java)
//启动多个Activity
ContextCompat.startActivities(this, arrayOf(intent1, intent2))
val options = ActivityOptions.makeScaleUpAnimation(view, 0, 0, view.width, view.height)
//添加转场动画启动多个Activity
ContextCompat.startActivities(this, arrayOf(intent1, intent2), options.toBundle())
//跳转单个Activity，并可携带转场动画参数
ContextCompat.startActivity(this,intent1,options.toBundle())
//获取应用路径的File对象，此路径存储了所有私有文件；一般情况下，应用程序不应该直接使用此路径，应该改为 Context#getFilesDir()，Context#getCacheDir(),Context#getDir(String,int)或其它存储API。
//返回结果为：/data/user/0/package-name
ContextCompat.getDataDir(this)
//返回多个sd卡下obb目录下的私有数据（该目录一般是游戏的数据包目录）
//返回结果为：/storage/emulated/0/Android/obb/package-name
ContextCompat.getObbDirs(this)
//返回多个sd卡下的该应用私有数据区的files目录
//返回结果为：/storage/emulated/0/Android/data/package-name/files
ContextCompat.getExternalFilesDirs(this, null)
//带type参数的返回结果为：/storage/emulated/0/Android/data/package-name/files/Music
ContextCompat.getExternalFilesDirs(this, Environment.DIRECTORY_MUSIC)
//返回多个sd卡下该应用私有数据库的缓存目录
//返回结果为：/storage/emulated/0/Android/data/package-name/cache
ContextCompat.getExternalCacheDirs(this)
//获取图片资源文件的Drawable对象
ContextCompat.getDrawable(this, R.mipmap.beautiful_girl)
//获取文字颜色状态值
ContextCompat.getColorStateList(this, R.color.button_text)
//获取色值
ContextCompat.getColor(this, R.color.purple_200)
//检查权限是否被授权；返回值android.content.pm.PackageManager#PERMISSION_GRANTED = 0；android.content.pm.PackageManager#PERMISSION_DENIED = -1。
ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE)
//获取不会备份到远程存储的应用程序文件
//返回结果为：/data/user/0/package-name/no_backup
ContextCompat.getNoBackupFilesDir(this)
//获取保存应用程序代码缓存目录文件。适合在运行时存放应用产生的编译或者优化的代码
//返回结果为：/data/user/0/package-name/code_cache
ContextCompat.getCodeCacheDir(this)
//创建设备保护的存储上下文对象，api <24，返回 null，一般用于用户锁屏状态下使用（Direct boot）模式。
ContextCompat.createDeviceProtectedStorageContext(this)
//判断应用是否支持 Direct boot 模式
ContextCompat.isDeviceProtectedStorage(this)
//获取主线程 Executor
ContextCompat.getMainExecutor(this)
//启动前台 service
ContextCompat.startForegroundService(this, intent)
//获取系统服务
ContextCompat.getSystemService(this, AlarmManager::class.java)
//获取系统服务名字
ContextCompat.getSystemServiceName(this, AlarmManager::class.java)
```

##### 2.ActivityCompat

```kotlin
//将权限兼容性方法委托给自定义类，ActivityCompat之后对权限的检查首先会检查委托中是否可以处理权限请求校验。
ActivityCompat.setPermissionCompatDelegate(ActivityCompat.PermissionCompatDelegate customDelefate)
//跳转Activity
ActivityCompat.startActivityForResult(this, intent, requestCode, null)
//ActivityResultContract 会调用 Activity.startIntentSender(IntentSender,Intent,int,int,int)。这个ActivityResultContract带有一个IntentSenderRequest,必须使用 IntentSenderRequest.Builder对其进行构造。
/**
 * @param fillInIntent 如果不为null，则将其作为参数提供给IntentSender#sendIntent
 * @param flagsMask 想要更改的原始IntentSender中的Intent标志
 * @param flagsValues flagsMask的值
 * @param extraFlags 总是为0
 * @param options 转场动画参数
 */
ActivityCompat.startIntentSenderForResult(@NonNull Activity activity,
            @NonNull IntentSender intent, int requestCode, @Nullable Intent fillInIntent,
            int flagsMask, int flagsValues, int extraFlags, @Nullable Bundle options)
//清除当前任务栈中自身以及所有与当前 Activity 具有相同`taskAffinity`的 Activity，如果当前任务栈中存在多种的 `taskAffinity`，只会清除那些相同的。
ActivityCompat.finishAffinity(this)
//过渡动画结束后关闭Activity
ActivityCompat.finishAfterTransition(this)
//获取当前Activity的调用者
ActivityCompat.getReferrer(this)
//根据id查找view，未找到则抛出异常：IllegalArgumentException
ActivityCompat.requireViewById<TextView>(this, R.id.button)
//设置转场动画，Activity#makeSceneTransitionAnimation时的监听，在被启动页面监听
ActivityCompat.setEnterSharedElementCallback(this, object : SharedElementCallback() {
    override fun onSharedElementStart(
        sharedElementNames: MutableList<String>?,
        sharedElements: MutableList<View>?,
        sharedElementSnapshots: MutableList<View>?
    ) {
        super.onSharedElementStart(
            sharedElementNames,
            sharedElements,
            sharedElementSnapshots
        )
        Log.d("TAG", "onSharedElementStart: start")
    }

    override fun onSharedElementEnd(
        sharedElementNames: MutableList<String>?,
        sharedElements: MutableList<View>?,
        sharedElementSnapshots: MutableList<View>?
    ) {
        super.onSharedElementEnd(sharedElementNames, sharedElements, sharedElementSnapshots)
        Log.d("TAG", "onSharedElementEnd: end")
    }
})
//设置转场动画，Activity#makeSceneTransitionAnimation时的监听，在启动页面监听
ActivityCompat.setExitSharedElementCallback(this, object : SharedElementCallback(){...})
//暂时延迟Transition的使用，直到确定了共享元素的确切大小和位置，与startPostponedEnterTransition配合使用
ActivityCompat.postponeEnterTransition(this)
//恢复Transition的过度效果，与postponeEnterTransition配合使用
ActivityCompat.startPostponedEnterTransition(this)
//请求授权
ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.READ_CONTACTS), 100)
//是否需要授权窗口，分各种情况。1.在允许询问时返回true；2.在权限通过，或者权限被拒绝并且禁止询问时返回false，但是有一个例外，就是重来没有询问过的时候，也是返回false，所以单纯地去判断是没有作用的，只能在请求权限回调后再使用。Google的原意是：1.没有申请过权限，申请就是了，所以返回false。2.申请了用户拒绝了，那你就要提示用户了，所以返回true。3.用户选择了拒绝并且不再提示，那你也不要申请了，也不再提示用户，所以返回false。4.已经允许了，不需要申请也不需要提示，所以返回false。
ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.READ_CONTACTS)
//创建 DragAndDropPermissionsCompat绑定到此活动的对象，并控制与 DragEvent关联的内容URI的访问权限。与多窗口实现相关
ActivityCompat.requestDragAndDropPermissions(this, DragEvent)
//重新创建 Activity
ActivityCompat.recreate(this)
```

##### 3.ActivityManagerCompat

```kotlin
//判断设备是够是一个低内存的设备，如果返回为true则建议开发者减少一些消耗内存的操作。Android规定运行内存小于512的设备为低内存设备
ActivityManagerCompat.isLowRamDevice(activityManager)
```

##### 4.ActivityOptionsCompat

```kotlin
//与overridePendingTransition类似，在页面之间做过度动画，注意退出的时候调用ActivityCompat.finishAfterTransition(this)才有动画效果。
val activityOptionsCompat = ActivityOptionsCompat.makeCustomAnimation(
    this,
    android.R.anim.slide_in_left,//进入动画
    android.R.anim.slide_out_right//退出动画
)
ActivityCompat.startActivity(
    this,
    Intent(this, SecondActivity::class.java),
    activityOptionsCompat.toBundle()
)
//放大一个view，进行转场动画
ActivityOptionsCompat.makeScaleUpAnimation(it, it.width / 2, it.height / 2, 0, 0)
//一个点圆形渐变到全部显示，参数含义与makeScaleUpAnimation一样，动画太快了，看不出和makeScaleUpAnimation的区别
ActivityOptionsCompat.makeClipRevealAnimation(it, it.width / 2, it.height / 2, 0, 0)
//和 makeScaleUpAnimation 的区别是，不再是放大页面上的一个 View，而是指定一张图，在转场时，放大这张图片。这个也看不见
ActivityOptionsCompat.makeThumbnailScaleUpAnimation(
    it,
    BitmapFactory.decodeResource(resources, R.mipmap.ic_launcher),
    0,
    0
)
//对单个view进行过渡动画,共享元素在xml中需要添加android:transitionName="share"
ActivityOptionsCompat.makeSceneTransitionAnimation(this, it, "share")
//对多个view进行元素共享过渡动画
ActivityOptionsCompat.makeSceneTransitionAnimation(
    this, Pair(it, "share"),
    Pair(textView, "shareTv")
)
//如果与Intent.FLAG_ACTIVITY_NEW_DOCUMENT一起设置，则不会向用户显示正在启动的任务，而只能通过“最近任务”列表使用。
val intent = Intent(this, SecondActivity::class.java)
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_DOCUMENT)
val activityOptionsCompat = ActivityOptionsCompat.makeTaskLaunchBehind()
ActivityCompat.startActivity(this, intent, activityOptionsCompat.toBundle())
//创建一个基本的ActivityOptions，它没有与之关联的特殊动画。其他options可以设置它。
val activityOptionsCompat =
    ActivityOptionsCompat.makeSceneTransitionAnimation(this, Pair(it, "share"))
val basicOptionsCompat =
    ActivityOptionsCompat.makeBasic()
basicOptionsCompat.update(activityOptionsCompat)//替代
```

##### 5.AlarmManagerCompat

```kotlin
//设置闹钟，在固定时间执行，在Android6.0之后，谷歌加入了新的Doze模式，即屏幕关闭后，系统会对CPU、网络、Alarm等活动做出限制，从而延长寿命。因此此方法不再准时，如果非要准时执行，请使用setAndAllowWhileIdle或setExactAndAllowWhileIdle
val activityIntent = Intent(this, SecondActivity::class.java)
val activityPendingIntent = PendingIntent.getActivity(this, 1, activityIntent, 0)
val broadcastReceiver = Intent(this, CustomBroadcastReceiver::class.java)
val broadcastPendingIntent = PendingIntent.getBroadcast(this, 0, broadcastReceiver, 0)
AlarmManagerCompat.setAlarmClock(
    alarmManager,
    System.currentTimeMillis() + 1000 * 10,
    activityPendingIntent,
    broadcastPendingIntent
)
//在新的Doze模式下依然能够准时执行
AlarmManagerCompat.setAndAllowWhileIdle(
    alarmManager,
    AlarmManager.RTC_WAKEUP,
    System.currentTimeMillis() + 1000 * 10,
    broadcastPendingIntent
)
//Android 4.4之后，Alarm任务的触发时间不再准确，这是系统做出了耗电优化，使用setExact或setExactAndAllowWhileIdle时时间再次准确
AlarmManagerCompat.setExact(
    alarmManager!!,
    AlarmManager.RTC_WAKEUP,
    System.currentTimeMillis() + 1000 * 10,
    broadcastPendingIntent
)
//在新的Doze模式下，并且忽略Android 4.4之后的耗电优化
AlarmManagerCompat.setExactAndAllowWhileIdle(
    alarmManager!!,
    AlarmManager.RTC_WAKEUP,
    System.currentTimeMillis() + 1000 * 10,
    broadcastPendingIntent
)
```

##### 6.AppLaunchChecker

```kotlin
//使用SharedPreferences记录App的使用状态，以供此类上的其它方法以后使用
AppLaunchChecker.onActivityCreate(this)
//使用上面的onActivityCreate后，可通过此方法校验此Activity是否启动过
AppLaunchChecker.hasStartedFromLauncher(this)
```

##### 7.AppOpsManagerCompat

```kotlin
//AppOps主要用于对权限的访问控制和电池消耗的跟踪。一般用于Android 6.0运行时权限管理机制（targetSDkVersion < 23）之前的权限管理。该类在API 19（Android 4.3）被引入。
//获取与给点权限关联的应用操作名称，返回android:read_contacts
AppOpsManagerCompat.permissionToOp(Manifest.permission.READ_CONTACTS)
//检测应用是否具有某项权限的操作，在校验之后会做记录，返回MODE_ALLOWED/MODE_IGNORED
val permissionToOp =
    AppOpsManagerCompat.permissionToOp(Manifest.permission.READ_CONTACTS)
val uid = packageManager.getApplicationInfo(application.packageName, 0).uid
val noteOp =
    AppOpsManagerCompat.noteOp(this, permissionToOp!!, uid, application.packageName)
//与noteOp一样，但是它不会抛出SecurityException
val noteOp =
    AppOpsManagerCompat.noteOpNoThrow(
        this,
        permissionToOp!!,
        uid,
        application.packageName
    )
//与noteOp一样，但是需要传递代理应用程序的应用程序名
val noteOp =
    AppOpsManagerCompat.noteProxyOp(
        this,
        permissionToOp!!,
        application.packageName
    )
//与noteOpNoThrow一样，但是需要传递代理应用程序的应用程序名
val noteOp =
    AppOpsManagerCompat.noteProxyOpNoThrow(
        this,
        permissionToOp!!,
        application.packageName
    )
```

##### 8.BundleCompat

```kotlin
//为所有Android版本获取IBinder(跨进程通信)的便利方法
BundleCompat.getBinder(bundle, "key")
//为所有Android版本将IBinder放入Bundle中的便利方法
BundleCompat.putBinder(bundle, "key", iBinder)
```

##### 9.DialogCompat

```kotlin
//查找Dialog中的view控件，如果view为null，抛出IllegalArgumentException
DialogCompat.requireViewById(dialog, R.id.button)
```

##### 10.NavUtils

```kotlin
//如果点击返回按钮时需要创建activity，则返回true
NavUtils.shouldUpRecreateTask(this, intent)
//跳转时携带FLAG_ACTIVITY_CLEAR_TOP标志，销毁目标Activity和它之上的所有Activity，重新创建目标Activity
val intent = Intent(this, MainActivity::class.java)
NavUtils.navigateUpTo(this, intent)
//仅当sourceActivity和parent Activity在同一任务栈中，才能使用此方法，否则抛出IllegalArgumentException
val intent = NavUtils.navigateUpFromSameTask(this)
//获取xml中设置的parent Activity
val intent = NavUtils.getParentActivityIntent(this)
val componentName = ComponentName(packageName,ThridActivity::class.java.name)
NavUtils.getParentActivityIntent(this, componentName)
NavUtils.getParentActivityIntent(this, ThridActivity::class.java)
//获取xml中设置的parent Activity的名字，返回heven.holt.jetpack.SecondActivity
NavUtils.getParentActivityName(this)
val componentName = ComponentName(packageName, ThridActivity::class.java.name)
NavUtils.getParentActivityName(this, componentName)
```

##### 11.NotificationManagerCompat

```kotlin
//创建一个通知
val notificationManagerCompat = NotificationManagerCompat.from(this)

val channelId = "channel_01"
val notificationChannel =
    NotificationChannel(channelId, "渠道名字", NotificationManager.IMPORTANCE_HIGH)
notificationChannel.description = "渠道描述"
notificationChannel.enableLights(true)
notificationChannel.lightColor = Color.RED
notificationChannel.enableVibration(true)

val notification = Notification.Builder(this, channelId)
    .setSmallIcon(R.mipmap.ic_launcher_round)
    .setContentText("his is content text")
    .setWhen(System.currentTimeMillis())
    .setAutoCancel(true)
    .setContentTitle("This is content title").build()

notificationManagerCompat.createNotificationChannel(notificationChannel)
notificationManagerCompat.notify(1, notification)

//根据id取消先前显示的通知
notificationManagerCompat.cancel(1)
//根据设置的tag与id取消通知
notificationManagerCompat.cancel("tag1", 1)
//取消所有通知
notificationManagerCompat.cancelAll()
//设置id显示通知
notificationManagerCompat.notify(1, notification)
//设置tag与id显示通知
notificationManagerCompat.notify("tag1", 1, notification)
notificationManagerCompat.notify("tag2", 1, notification)
//检测通知是否可用；true-可用；false-不可用。
notificationManagerCompat.areNotificationsEnabled()
//获取显示通知的重要性。比如IMPORTANCE_DEFAULT=-1000
notificationManagerCompat.importance
//创建通知渠道；api>26
notificationManagerCompat.createNotificationChannel(notificationChannel)
//创建通知渠道组，对于多种用户的App，比如社交应用有工作账号和私人账号，可以将定义不同的用户下所有的channel为一个group进行管理。
val notificationChannelGroup = NotificationChannelGroup("channel_id","group")
notificationChannelGroup.channels.add(notificationChannel)
notificationManagerCompat.createNotificationChannelGroup(notificationChannelGroup)
//创建多个通知渠道
notificationManagerCompat.createNotificationChannels(notificationChannels)
//创建多个通知渠道组
notificationManagerCompat.createNotificationChannelGroups(notificationChannelGroups)
//根据渠道id移除渠道，也会移除通知
notificationManagerCompat.deleteNotificationChannel("channel_01")
//根据渠道组移除渠道组，也会移除通知
notificationManagerCompat.deleteNotificationChannelGroup("channel_group")
//根据channelId获取通知渠道
notificationManagerCompat.getNotificationChannel("channel_01")
//根据channelGroupId获取通知渠道组
notificationManagerCompat.getNotificationChannelGroup("channel_id")
//返回所有的渠道：List<NotificationChannel>
notificationManagerCompat.notificationChannels
//返回所有的渠道组：List<NotificationChannelGroup>
notificationManagerCompat.notificationChannelGroups
//获取已开启权限的包名从而判断通知是否开启，可以直接使用areNotificationsEnabled替代，但是在测试中，这个函数总是返回空集合。
NotificationManagerCompat.getEnabledListenerPackages(this)
```

##### 12.NotificationCompat

```kotlin
//从notification中获取extra；extra为Notification内部已经实例化的Bundle,Content等相关信息存储在这里。
val bundle = NotificationCompat.getExtras(notification)
//获取通知中的Action数量
NotificationCompat.getActionCount(notification)
//根据index获取Action
NotificationCompat.getAction(notification, 0)
//获取气泡通知（BubbleMetadata），该通知将用于在现有前台活动上方的浮动窗口中显示内容
NotificationCompat.getBubbleMetadata(notification)
//获取通知中不可见的操作
NotificationCompat.getInvisibleActions(notification)
//获取通知标题
NotificationCompat.getContentTitle(notification)
//获取通知类别
NotificationCompat.getCategory(notification)
//获取通知是否仅与当前设备相关。某些通知可以桥接到其它设备以进行远程显示。如果设置了此提示，建议不要桥接此通知
NotificationCompat.getLocalOnly(notification)
//获取用于此通知分组到群集或堆栈的密钥，以及与支持此类呈现的设备上的其它通知一起使用
NotificationCompat.getGroup(notification)
//获取此通知是否为一组通知的主摘要
NotificationCompat.isGroupSummary(notification)
//获取排序键，该排序键可在同一包中的其他通知中对该通知进行排序。如果外部程序通知希望保留排序，则需要用到此值
NotificationCompat.getSortKey(notification)
//获取此通知的渠道id
NotificationCompat.getChannelId(notification)
//返回应用应取消此通知的时间，不取消则返回0
NotificationCompat.getTimeoutAfter(notification)
//返回通知的桌面角标类型；BADGE_ICON_NONE，BADGE_ICON_SMALL，BADGE_ICON_LARGE
NotificationCompat.getBadgeIconType(notification)
//返回通知的快捷方式id
NotificationCompat.getShortcutId(notification)
//返回通知组以哪种声音方式提醒用户：GROUP_ALERT_ALL，GROUP_ALERT_CHILDREN，GROUP_ALERT_SUMMARY
NotificationCompat.getGroupAlertBehavior(notification)
//返回是否允许开发人员为此通知生成上下文操作
NotificationCompat.getAllowSystemGeneratedContextualActions(notification)
```

##### 13.RemoteActionCompat

```kotlin
//主要用于RemoteActionCompat与RemoteAction之间的转化，API26+
val intent = Intent(this, SecondActivity::class.java)
val pendingIntent = PendingIntent.getActivity(this, 0, intent, 0)
val remoteAction = RemoteAction(
    Icon.createWithResource(this, R.mipmap.beautiful_girl),
    "remoteTitle",
    "remoteDescription",
    pendingIntent
)
RemoteActionCompat.createFromRemoteAction(remoteAction)
//反转
 createFromRemoteAction.toRemoteAction()
```

##### 14.ServiceCompat

```kotlin
//停止前台服务，flag:STOP_FOREGROUND_REMOVE,前台通知将被移除。STOP_FOREGROUND_DETACH，前台通知将被与service分离，该通知保持显示状态，但是会与service完全分离，因此不再更改，除非再次调用，此标志仅在api24及以上生效。
ServiceCompat.stopForeground(service, flag)
```

##### 15.ShareCompat

```kotlin
//从共享Inteng中获取启动分享的程序包名称，提供社交分享功能的应用可以使用此功能归纳共享内容的应用程序
ShareCompat.getCallingPackage(this)
ShareCompat.getCallingActivity(this)
//配置menu为共享菜单
ShareCompat.configureMenuItem(menuItem, shareIntent)
ShareCompat.configureMenuItem(menu, menuId, shareIntent)
```

##### 16.ContentResolverCompat

```kotlin
//查询媒体库
val projection = arrayOf(
    MediaStore.Images.Media._ID,
    MediaStore.Images.Media.DISPLAY_NAME,
    MediaStore.Images.Media.DATA
)
//查询媒体库
val query = ContentResolverCompat.query(
    contentResolver, MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
    projection, null, null, null, null
)

val id = query.getLong(0)
val uri =
    ContentUris.withAppendedId(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, id)
imageView.setImageURI(uri)
```



