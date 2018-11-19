# Android基础知识 
## 1.Android应用原理
### Android app运行在各自的安全沙盒里
* Android系统是一个多用户的Linux系统，每个app是一个不同的用户
* Android系统给每一个App分配一个用户id（这个用户id只有android系统可见，App本身并不知道）；系统给app的文件设置访问权限，所以只有该App本身才有权限去访问这些问题
* 每个进程对应一个虚拟机，所以说每个App运行空间是和其他App空间隔离的
* 默认情况下，每个App运行在各自的Linux进程中。当运行App中的任何一个组件时，该App进程就会运行起来
### 启动Android组件
* activities，services和broadcastReceiver是通过一个异步的Intent来启动的
* Content Provider是通过ContentResolver来启动的
### Manifest文件
* 在Android系统启动一个组件之前，系统通过读取manifest文件来得知这个应用包括了哪些组件，所有的组件都要在这个文件中申明
* 定义这个应用需要使用的权限
* 定义这个应用支持的api版本
* 定义这个应用支持的软件和硬件特性

## 2.Android性能优化
[Android 性能优化最佳实践](https://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649492955&idx=1&sn=fea32aad2214cd448b584b74a06f7934&pass_ticket=cjZPjE07iJuMH4Luv%2FvlLFi4d9kowFjv84o%2FPPJ4GETsu4ATP0ulZs343ouiG31r)
### 1). 布局优化
* 如果父控件有颜色，也是自己需要的颜色，那么就不必在子控件加背景颜色（这个应该是绘制的时候才会有影响）
* 如果每个自控件的颜色不太一样，而且可以完全覆盖父控件，那么就不需要再父控件上加背景颜色（这个应该是绘制的时候才会有影响）
* 尽量减少不必要的嵌套
* 能用LinearLayout和FrameLayout，就不要用RelativeLayout，因为RelativeLayout控件相对比较复杂，测绘也想要耗时。？？？
* 使用include和merge增加复用，减少层级
* ViewStub按需加载，更加轻便
### 2). 绘制优化
卡顿：一帧（16ms）内不能绘制不能完成，就会导致后面的一帧会延后，或者直接略过，这样就会卡顿了
* onDraw方法中不要做耗时的任务，也不做过多的循环操作，特别是嵌套循环，虽然每次循环耗时很小，但是大量的循环势必霸占CPU的时间片，从而造成View的绘制过程不流畅。
* 除了循环之外，onDraw()中不要创建新的局部对象，因为onDraw()方法一般都会频繁大量调用，就意味着会产生大量的零时对象，不仅占用过多的内存，而且会导致系统更加频繁的GC，大大降低程序的执行速度和效率。
### 3). 内存优化
因为有内存泄漏，所以内存被占用越来越多，那么GC会更容易被触发，GC会越来越频发，但是当GC的时候所有的线程都是暂停状态的，需要处理的对象数量越多耗时越长，所以这也会造成卡顿。
内存泄漏
* 集合类泄漏
* 单例/静态变量造成的内存泄漏
* 匿名内部类/非静态内部类
* 资源未关闭造成的内存泄漏
1). 网络、文件等流忘记关闭
2). 手动注册广播时，退出时忘记 unregisterReceiver()
3). Service 执行完后忘记 stopSelf()
4). EventBus 等观察者模式的框架忘记手动解除注册
### 4). 启动速度优化
* 利用提前展示出来的Window，快速展示出来一个界面，给用户快速反馈的体验；
* 避免在启动时做密集沉重的初始化（Heavy app initialization）；
* 避免I/O操作、反序列化、网络操作、布局嵌套等。
### 5). 包优化
* 删除不用的资源
* 使用xml drawable代替切图
* 重用资源
* 代码混淆
### 6). 耗电优化
### 7). ListView和Bitmap优化
* ListView使用ViewHolder，分段，分页加载
* 对图片质量进行压缩 
* 对图片尺寸进行压缩 
* 使用libjpeg.so库进行压缩
### 8). 响应速度优化
* 避免在主线程中做耗时操作，把耗时操作异步处理
### 9). 线程优化
* 减少在创建和销毁线程上所花的时间以及系统资源的开销
* 如不使用线程池，有可能造成系统创建大量线程而导致消耗完系统内存以及”过度切换”
### 10). 其他优化
## 3. ListView的缓存机制
[源码解析ListView中的RecycleBin机制 - CSDN博客](https://blog.csdn.net/iispring/article/details/50967445)
[Android ListView工作原理完全解析，带你从源码的角度彻底理解 - CSDN博客](https://blog.csdn.net/guolin_blog/article/details/44996879)
### RecycleBin
* mActiveVIews: 这个是数组，里面保存着当前显示在屏幕中的View， 只在onlayout阶段才用到，其他阶段数组为空；其主要目的是为了优化layout的速度。mActiveViews取出的view不需要重新创建，也不需要重新bind数据，所以可以提升layout的速度。
* mScrapViews：这个是数组，数组的元素个数跟ViewType的个数相同，每个数据元素是一个List<View>。其主要目的是为了缓存被移出屏幕的View，提供给Adapter的getView(int position, View convertView, ViewGroup parent)使用，这里covertView就是缓存view。这也就意味着，从mScrapViews取出的view不需要重新创建，不过需要重新bind一下数据
## 4. RecyclerView
[Android ListView 与 RecyclerView 对比浅析：缓存机制 - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1005658)
[Building a RecyclerView LayoutManager – Part 1 | Wires Are Obsolete](http://wiresareobsolete.com/2014/09/building-a-recyclerview-layoutmanager-part-1/)
[Anatomy of RecyclerView: a Search for a ViewHolder – AndroidPub](https://android.jlelse.eu/anatomy-of-recyclerview-part-1-a-search-for-a-viewholder-404ba3453714)
[探究RecyclerView的ViewHolder复用 - 简书](https://www.jianshu.com/p/d7ec36aa8e4b)
[深入浅出 RecyclerView|开源实验室](https://www.kymjs.com/code/2016/07/10/01/)
### Recycler
* LayoutManager持有一个Recycler，当LayoutManager需要回收旧的Views时，或者需要获取新的Views，这些工作交由Recycler来实现。
* LayoutManager需要保证当Views不可见时，把它们交由Recycler处理。
* 在Layout过程中，有两种方式来处理已经存在的Views，detach和remove。
* Detach是一个轻量级的操作，可能用来将Views重新排序，所有detached Views可能还会attached回来。这个操作只会修改Views索引，但不会导致Views的重建和重新绑定。
* Remove意味着Views是不需要的了，所以这些Views会放在Recycler里面供后面使用
* mAttachedScrap和mCachedViews是轻量级的缓存系统，从这里面取出的ViewHolder不需要重新绑定数据。它们是一个List，不是按ViewType区分。
* mRecyclerPool则保存一些Views（按ViewType区分），如果从mRecyclerPool取出Views，则需要重新绑定数据。这是因为如果一个View进入了mRecyclerPool，其他的状态都会被清除（其中最重要的是position），只会保留View本身和ViewType。
* 当LayoutManager需要一个新的View时，先根据position/id从mAttachedScrap和mCachedViews查找，如果存在，则直接返回这个View并且不用重新绑定数据；如果不存在，则从mRecyclerPool取出一个正确ViewType的View，并且重新绑定数据；如果mRecyclerPool不存在缓存的View，那么调用Adapter.createViewHolder()创建View，然后再绑定数据。这里需要区分的就是从mAttachedScrap和mCachedViews查找，是通过position的，而在mRecyclerPool中查找，则是通过ViewType的。
* 如果找不到View，那么需要创建，然后绑定数据；如果从Pool里找到了View，则需要绑定数据；如果从cache里找到View，则不需要绑定数据

## 应用权限
* 罗列在manifest的普通权限会被系统自动的授权
* 罗列在manifest的危险权限（不安全的，会影响用户隐私的）则需要用户授权之后才能够使用（Android 6.0 - API 23）；Android 6.0后，需要运行时申请危险权限。如果低于Android6.0，则安装时需要用户确认是否授权
* 对于activities和services，如果有权限限制，并且启动对应的activities或者activities的调用者没有对应的权限，则会抛出SecurityException
* 对于Broadcast Receiver，如果调用者没有对应的权限，不会抛出SecurityException，只不过不会把对应的Intent发出去
* Content provider 的访问权限用来限制调用者，Content Provider包括读权限和写权限，拥有了写权限不意味着你有读权限。如果没有相对应的权限，则会抛出SecurityException
* 申请属于一个Permission group的权限时，比如申请READ_CONTACTS 权限时，申请对话款会提示申请CONTACTS权限，并不会指明READ_CONTACTS权限；如果READ_CONTACTS权限已经授权，那么再申请WRITE_CONTACTS权限，此时不会弹出对话框，系统会直接授权，因为它与READ_CONTACTS属于一个Permission group。

## App Manifest
这个文件提供一些必要的信息给Android build tools，Android操作系统以及Google player
* 定义了包名，Android编译工具在编译代码时会使用这个包名来确定代码的位置；在App打包时，编译工具会使用gradle文件的application id来代替这个包名，在android系统和google player里使用application id来唯一识别该App
* 定义所有的组件，包括名称，device configuration，intent filer以及组件的启动模式
* 定义了该app需要使用系统和其他app的权限，以及其他app使用这个app所需要的权限
* 定义了这个app所需的软件以及硬件需求
### Package name的用途
* 一个是用来当作资源文件（即com.example.myapp.R）的命名空间
* 一个是用来解析定义在manifest文件中的相对类名（.MainActivity，这个就会被解析为com.example.myapp.MainActivity）
### 版本名的作用
minSdkVersion，定义运行这个app所需的最小sdk版本号
targetSdkVersion， 定义这个app是在那个版本测试的

## App资源
### 资源类型
animator: 属性动画
anim: 补间动画
color： 用来定义状态相关的颜色
drawable
mipmap
layout
menu
raw: 通过Resources.openRawResources(R.raw.filename)来访问， 如果资源需要通过文件层级来访问，那么请放在assets目录中，并通过AssetManager来访问
values： 可以定义arrays, colors, dimens,strings以及styles
xml： 通过Resources.getXml()来访问
font：字体文件如.ttf,otf或者.ttc
### 资源修饰符
* 可以给一个资源目录以多个资源修饰符，不过他们的顺序需要遵守规则，修饰符之间通过“-”符号来分隔开
* 资源文件夹的名称不区分大小写
* 对于每种类型的资源修饰符只能有一个值，如可以有strings-en,但不可以有 string-en-fr
### 资源引用
可以通过资源引用来减少资源的空间占用
### 资源访问
* 引用xml资源：java，R.string.name; xml, @string/name
* 引用style属性：?android:textColorSecondary
* assets : AssetManager
* Raw: Resources.openRawResources()
### 资源适配
为了App更好的支持不同的配置，需要提供为各种不同的资源提供默认的资源

## 处理配置变化
* 有一些设备配置会在运行的过程中发生变化，如屏幕方向，键盘可用，多窗口等，这些配置变化发生时，会导致activity的重建(onDestroy() & onCreate())。
* 如果不想因为转屏而导致activity重建，则需要在manifest文件中指定android:configChanges为“orientation|screenSize”。
## App本地化
默认资源需要全部提供
本地化资源可以只提供部分
## App资源类型
### 1.Animation Resources（动画资源）
Define pre-determined animations
* Tween animations are saved in res_anim_ and accessed from the R.anim class.
* Frame animations are saved in res_drawable_ and accessed from the R.drawable class.
#### 属性动画
res_animator_filename.xml
In Java: R.animator.filename
In XML: @[package:]animator/filename
通过在一定时间内不断的修改对象的属性值来创建一个动画
#### View动画
有两类动画可以通过View动画框架来呈现
补间动画：对一个图片进行一系列的转换（包括旋转、平移、扭曲以及渐变）来创建一个动画
In Java: R.anim.filename
In XML: @[package:]anim/filename
逐帧动画：通过不断的显示一个图片序列来创建一个动画
In Java: R.drawable.filename
In XML: @[package:]drawable.filename
### 2.Color State List Resource（颜色状态列表资源）
Define a color resources that changes based on the View state.
* Saved in res_color_ and accessed from the R.color class.
### 3.Drawable Resources（图形资源）
Define various graphics with bitmaps or XML.
Saved in res_drawable_ and accessed from the R.drawable class.
1. BitmapDrawable(.png,.jpg,.gif) ：可以通过直接应用图片，或者在xml定义一个图片引用；图片在编译的时候有可能会被优化，以此来减少内存的使用
In Java: R.drawable.filename
In XML: @[package:]drawable/filename
2. NinePathDrawable(.9.png)
3. LayerDrawable：管理一个Drawable数组，它是按照Drawable的顺序来绘制的，最大索引的Drawable画在最上层。
4. StateListDrawable：在一个xml文件中，定义不同状态下显示不同的图片
5. LevelListDrawable：通过一个xml文件定义一个Drawable，这个Drawable管理一定数量可交替的Drawable，每个Drawable定义一个最大的数值。
6. TransitionDrawable：在一个xml文件中定义一个drawable，它可以在两个drawable资源中进行状态转变
7. InsetDrawable：通过一个xml文件来定义一个drawable，这个drawable可以在另一个drawable的边上插入间隔，主要用在一个view只需要一个背景比它自己还小时使用
8. ClipDrawable：通过一个xml文件来定义一个drawable，这个drawable通过当前的等级值来裁剪另一个图片
9. ScaleDrawable：通过一个xml文件来定义一个drawable，这个drawable通过当前的等级值来改变另一个drawable的大小
10. GradientDrawable：通过一个xml来定义物理图形，包括颜色和渐变
11. AnimationDrawable：
注意：一个颜色资源可以当作drawable资源来使用
### 4.Layout Resource
Define the layout for your application UI.
* Saved in res_layout_ and accessed from the R.layout class.
### 5.Menu Resource
Define the contents of your application menus.
Saved in res_menu_ and accessed from the R.menu class.
### 6.String Resources
Define strings, string arrays, and plurals (and include string formatting and styling).
Saved in res_values_ and accessed from the R.string, R.array, and R.plurals classes.
* String
* String Array
* Quantity Strings(plurals)
#### 处理特殊字符
@	\@
?	\?
<	&lt;
&	&amp;
### 7.Style Resource
Define the look and format for UI elements.
Saved in res_values_ and accessed from the R.style class.
样式资源可以应用在View，Activity以及Application上
### 8.Font Resources
Define font families and include custom fonts in XML.
Saved in res_font_ and accessed from the R.font class.
### 9.More Resource Types
Define other primitive values as static resources, including the following:
#### 1).Bool
XML resource that carries a boolean value.
#### 2).Color
XML resource that carries a color value (a hexadecimal color).
#### 3).Dimension
XML resource that carries a dimension value (with a unit of measure).
#### 4).ID
XML resource that provides a unique identifier for application resources and components.
#### 5).Integer
XML resource that carries an integer value.
#### 6).Integer Array
XML resource that provides an array of integers.
#### 7).Typed Array
XML resource that provides a TypedArray (which you can use for an array of drawables).

## Activity简介
* 一个Activity就是一个应用的入口
* 一个Activity提供一个窗口供应用来绘制UI
* 多个Activities之间是松耦合的
什么东西是不可更改的？
1).应用的包名和应用的签名：如果改变了就是一个新的应用，那样就不能进行应用升级（如果变更了包名，那么会在一个设备上安装新旧两个版本的应用；如果变更了签名，除非删除旧的应用，否则新的应用不能安装，也就是说如果变更签名，那么包名也需要变更，否则应用将安装失败）
2). AndroidMenifest.xml是一个public的API：
### 1.理解Activity的生命周期
Android 7.0（API level24），多个app可以运行在多窗口模式，但是只有一个app可以获取焦点
* 不要在onPause()做应用和用户数据保存，网络请求以及数据库操作，因为onPause执行是非常短暂的
* 在onStop()状态时，activity的状态以及成员信息还保存在内存中，但是没有attach在WindowManager上
* onRestoreInstanceState()在onStart()之后调用，并且只有有保存数据的时候才会被调用
#### 1).Activity状态和内存释放
系统一般不会杀死一个activity来释放内存，而是杀死activity所在的进程来释放内存
#### 2).Activity的瞬间UI状态保存和恢复
如果UI数据比较简单并且轻量，那么使用onSaveInstanceState()就可以了；否则，你需要同时使用ViewModel和onSaveInstanceState()
#### 3).Activities的跳转
startActivityForResult的requestCode不适全局的id，所以在不同的activity使用相同的requestCode并没有什么关系
### 2.理解任务和回退栈
一个任务就是Activities的集合
在Android 7.0（API 24）以上，如果是多窗口环境，那么系统为每个窗口分别管理任务栈（每个窗口可能存在一个或者多个任务栈）
#### 管理任务
* 有一些启动模式只在manifest可用，有些则只在intent的flag中可用
* 如果通过intent设置了一个启动flag，并且目标activity在manifest也设置了启动模式，那么以intent设置的flag为准
1. Manifest的启动模式：standard，singleTop，singleTask，singInstance
(1).singleTop：如果任务栈为A->B->C，并且C为singleTop，那么如果再启动C，此时onNewIntent可以得到调用，但是如果点击后退键，则不会回退到之前的A->B->C状态，而是A->B
(2).singTask：创建一个新的task，并且在这个task上创建一些新的activity，但是如果已经有一个activity存在，并且这个activity存在于一个分开的task上，那么此时不会创建一个新的task和一个新的activity，只会调用onNewIntent()
(3).singleInstance:跟singleTask类似，不过这个task只会存在一个activity
不管启动一个activity是在一个新的task或者在原来的 task，点击后退按钮会回退到之前的那个activity
2. Intent的flag：FLAG_ACTIVITY_NEW_TASK，FLAG_ACTIVITY_SINGLE_TOP，FLAG_ACTIVITY_CLEAR_TOP，

### 3.进程和应用生命周期

* 前台进程（foreground process）
activity onResume()，BroadcastReceiver onReceive()，以及service的onCreate(), onStart(), onDestroy()
* 可见进程（visible process）
activity onPause（），service.startForegound()，
* 服务进程 （service process）
* 缓存进程（cached process）

### 4.Fragment
* 一个fragment一般都运行在activity里，fragment的生命周期受activity生命周期的影响
* fragment是在android3.0（API 11）中引入的，主要为了支持更加灵活和动态 的UI设计
* 在使用fragment的时候需要考虑模块化和可重用性
#### 1).创建Fragment
在layout文件中引用fragment
在代码中动态添加fragment到container中
#### 2).管理Fragment
通过findFragmentById或者findFragmentByTag来得到已经存在的fragment
通过代码popBackStack()来让fragment出栈
#### 3).执行Fragment操作
* 可以执行的操作包括add，replace，remove等操作，最后需要commit
* 如果一个执行了add，replace操作，然后一起commit，并且将这个transaction放入回退栈，那么点击back按钮会导致add，replace操作同时回退
* 执行commit操作需要在activity保存状态之前执行
#### 4).Fragment和activity的通信
* 通过activity实现fragment的接口来实现fragment和activity进行通信
* Fragment可以添加OptionsMenu和ContextMenu
* fragment的getContext在fragment还没有attach或者已经detach时返回的是null
* Fragment之间的通信一般通过所在的activity来进行
### 5.和其他App交互
* 通过使用implicit intent启动其他app
* 通过startActivityForResult来获取从其他activity得到数据
* 不管本activity是否通过startActivityForResult启动，都可以通过setResult来设置返回结果；如果不是通过startActivityForResult启动的activity，那么这个结果被忽略

### 处理android app的链接

## Intents和Intent Filters
### 1.Intent类型
显式Intent：通过包名或者组件名来指定一个组件
隐式Intent：通过action，data，type，category来匹配一个component；为了能够接收到隐式Intent，需要在Intent filter里定义Category default。
### 2.PendingIntent
* PendingIntent对象是一个包装的Intent对象，其主要目的是为了给其他应用授权使用该PendingIntent包含的Intent。简单的说就是这个PendingIntent包含的Intent不是给本应用使用的，而是由其他应用使用的。
* PendingIntent包含如下使用场景：点击通知从而启动某个Intent，这个Intent是由NotificationManager执行的；点击桌面app icon来启动该应用，这个Intent是由Home Screen 应用来执行的；通过AlarmManager来执行一个intent。
### 3.公用的Intents
Alarm Clock
Calendar
Camera
Contacts/People App
Email
File storage
Local Actions
Maps
Music or Video
New Note
Phone
Search
Settings
Text Messaging

## 动画
### 1.属性动画
1）工作模式：
创建一个ValueAnimator，然后给它一个起始时间和结束时间，以及一个属性的变化范围；启动这个动画之后，ValueAnimator会计算已经消逝的时间片，然后通过插值器来计算出一个0～1的值（这个值是由插值器的类型来决定的，如果是线性插值器，那么这个值和消逝的时间片一致），最后通过估值器以及起始终止属性值来得到对应的属性值，然后通过重新绘制这个对象从而得到相应的动画效果。
估值器：告诉属性动画系统如何计算属性的值。输入包括Animator提供的插值，以及动画的开始值和结束值，然后计算出属性值
插值器：输入是时间，输出的是插值，具体返回值跟插值器的类型相关
2）View动画和属性动画的不同点：
* View动画只适用在View上，而属性动画则不然
* View动画只能做scale，tranlation，fade以及rotate动画，而属性动画则可以改变很多View属性
* View动画只改变了View的绘制位置，但是实际的layout位置没有发生变化
* View动画更简单，所需的代码更少
3）一个属性动画的正确执行，需要如下条件
* 对应的属性需要有set方法，如setFoo
* 对应的属性需要get方法，如getFoo
* ObjectAnimator的开始值和结束值类型需要和对应的getter和setter方法对应的值类型一致
* 对于某些属性执行属性动画，需要手动调用invalidate方法去重绘这个view
### 2.图片动画
* AnimationDrawable
* AnimatedVectorDrawable？？？
### 3.使用动画来显示和隐藏View
在一个界面切换到另一个界面时，伴随着一个界面的显示和另一个界面的隐藏。如果是快速切换，则会显得有些突兀。而使用动画则会显得更加自然。
有三种动画用来显示和隐藏View
* 淡入淡出动画：使用的是ViewPropertyAnimator？？？
* 卡片切换动画：在Fragment切换的时候使用自定义动画？？？
* 建立一个环形动画：使用ViewAnimationUtils
### 4.使用动画来移动View
使用属性动画来移动View
使用曲线移动？？？
### 5.使用fling动画来移动View
FlingAnimation
### 6.使用Zoom动画来放大一个View
使用的是属性动画集合
### 7.使用spring physics来做动画
### 8.在layout变化时使用动画
设置android:animateLayoutChanges为true即可，如果想自定义动画，则自定义后通过setLayoutTransition方法即可
### 9.使用transition来实现layout变化动画（在layout变化的时候可以使用）
transition框架包括如下特性：
* 组级别动画：可以在一个View层级中使用一个或者多个动画效果
* 有一些默认动画
* 可以通过资源文件来加载View layout和默认动画
* 在动画和View层级变化时提供回调
Layout变化动画实现过程如下：
* 为起始layout和结束layout都创建一个Scene对象
* 定义一个transition对象，这个对象主要定义使用的动画类型
* 调用TransitionManager.go来实现layout变化动画
### 10.创建一个自定义transition动画
### 11.启动activity使用动画
动画主要用处：一个是显示和隐藏View的时候，一个是View移动的时候，一个是Layout变化的时候，一个是activity切换的时候

## Images and graphics
* Drawable是一个抽象的概念，表示一个可以被绘制的东西
* 在一个工程里每个资源只有一个状态。也就是说，如果你在两个不同的地方使用同一个Drawable资源，然后在其中一个地方改变资源的属性，比如说alpha值，那么另一个地方使用的资源alpha值也会被改变
* 可以给BitmapDrawable，NinePatchDrawable以及VectorDrawable使用setTint方法

### 1.矢量图
* 矢量图定义在xml文件中，通过点，线，弧线以及颜色来定义一个图片
* 使用矢量图的好处在于它不需要使用多个屏幕密度的图片，只需要一个xml文件就行
* 矢量图定义一个静态的图片，它定义成树结构，包括path以及group，path是叶子结点，而group是中间节点；path描述集合图形，而group描述变化

### 2.处理bitmap
加载bitmap会导致很多问题
* bitmap的一个像素点会占用4个字节（ARGB_8888）
* 在UI线程加载图片可能会导致ANR
* 如果你的应用需要加载多个bitmap，请做好内存以及磁盘缓存，不然你的应用响应会显得非常慢

### 3.从调色板选择颜色

### 4.减小图片的下载大小

### 5.硬件加速
* 在android3.0之后，支持硬件加速，也就是在View canvas上做的所有绘制操作都是通过GPU
* 并不是所有的2D绘制操作都支持硬件加速，所以android系统提供一个开关来打开和关闭硬件加速
硬件加速控制
* Application
* Activity
* Window：在Window级别可以开启硬件加速，但不可以关闭硬件加速
* View：在view级别不能启动硬件加速，不过可以关闭硬件加速
判断View是不是硬件加速·
* View.isHardwareAccelerated()：看看这个View是不是attached到一个硬件加速的window上
* Canvas.isHardwareAccelerated() ：看看这个canvas是不是硬件加速的
Android绘制模型

## 后台任务
### 1.如何使用后台任务能够不太影响应用性能？
每个Android应用都有一个UI线程，在UI线程上可以进行用户交互，接收生命周期回调；但是如果在UI线程做过多的事情，会导致应用变慢卡顿；所有的长时间操作如bitmap的decoding，访问磁盘以及网络请求都需要在子线程进行。

### 2.在使用后台任务时，请考虑如下因素
* 这个任务能够推延吗？是不是需要在安排时直接运行？比如网络请求
* 如果这个任务开始执行了，android os需要保证这个进程一直存活吗？比如bitmap decoding只有运行在前台时才执行；而音乐播放器则需要保持一直运行
* 这个任务需要在接收到系统调用后才运行吗？比如你想在飞行模式结束后才和服务器进行通信

### 3.后台任务的运行方式
* 如果一个任务需要进程在前台时才运行，请使用ThreadPools；
* 如果一个任务需要执行完成，并且你需要它立即执行，那么请使用前台服务
* 如果一个任务需要执行完成，不过它可以被推延，那么使用WorkManager（包括JobScheduler， FireBase JobDispatcher以及AlarmManager，SyncAdapter）

### 4.智能安排任务
JobScheduler：它是由android系统层实现的，它收集所有app的任务信息，通过这些任务信息安排任务执行，这样可以将这些工作在同一时间执行，从而延长电池的使用时间
AlarmManager：这个Api也是由android系统提供的，通过它你可以安排一些任务，由系统安排执行；如果一个任务需要在指定的时间去执行，那么请使用AlarmManager
Firebase JobDispatcher：一个开源库，类似JobScheduler，请在Android5.0之前使用
syncAdapter：是由Framework层提供的，它的主要目的是用来同步设备和云之间的数据，你只有在这种情况下才使用它
Service：推荐使用前台服务，绑定服务，减少使用startService

### 5.服务
有三种类型的服务：前台服务，后台服务，绑定服务
基础知识
onStartCommand：如果使用startService，那么就会调用这个回调，如果使用bind，那么这个方法可以不实现；调用startService启动服务，则需要调用stopSelf或者stopService来停止服务
onBind：调用bindService，则会调用这个回调；
onCreate：这个回调在服务启动的时候调用一次，并且在服务运行期间只会调用一次
onDestroy：当服务不再使用的时候调用
如何创建服务
创建IIntentService子类
* 创建一个子线程来执行任务
* 会创建一个工作队列保存任务列表
* 在所有请求结束后会自动结束服务
* 默认没有实现onBind
* 默认实现了onStartCommand
* 需要实现onHandleIntent
创建Service子类
onStartCommand的返回值
* START_NOT_STICKY：如果系统在onStartCommand之后杀掉这个服务，除非有pending的intent，否则不会重建这个服务
* START_STICKY：如果系统在onStartCommand之后杀掉这个服务，那么会重建这个服务，不过不会重新发送最后的intent，而是会使用一个null intent重新调用onStartCommand，当时如果有一个pending的intent，则会使用这个intent去调用onStartCommand
* START_REDELIVER_INTENT：如果系统在onStartCommand之后杀掉这个服务，那么系统会使用最后的intent去调用onStartCommand，其他pending的intent会依次被调用
* 多次调用startService会多次调用onStartCommand
* stopSelf（id）：如果这个id不是最新的id，那么则不会销毁这个服务
创建一个bound服务
给用户发通知
包括Toast notification和status bar notification

### 6.创建后台服务
IntentService有哪些限制？
* 不能直接操作UI
* 工作是依次执行的，不能并发执行
* 每一个工作都不能取消
JobIntentService？？？
### 7.创建绑定服务
多个客户端可以同时绑定到一个服务上，但是onBind只在第一个bindService时调用，其他客户端连接时直接返回这个iBinder
如何创建binder接口？
* 直接继承实现Binder类，如果这个service只在同一个应用中使用，并且在同一个进程
* 使用messenger：可以实现跨进程通信，而且由于messenger是在一个线程中序列调用的，所以不需要考虑线程安全问题？？？
* 使用AIDL：可以实现多线程， 不过需要考虑线程安全问题
只有activity，service以及content provider可以和服务绑定，而broadcast Receiver则不能

### 8.AIDL
定义客户端和服务端都遵守的api来进行进程间通信.
如何实现AIDL？
创建.aidl文件
定义IBinder实现
实现ServiceConnection
调用bindService
实现onServiceConnected
断链unBindService

### 9.后台优化
如果一个应用使用过度的资源，那么系统会提示用户去限制应用对资源的使用
Android 7.0之后，CONNECTIVITY_ACTION定义在manifest中将不起作用，但是动态注册还是有效的；你可以使用ConnectivityManager的registerNetworkCallback()来实现；或者使用JobScheduler来实现

## 广播
系统广播
什么是隐式广播？不指定应用的广播都是隐式广播
在代码中可以注册隐式广播和显式广播
注册广播：
* 如果是通过manifest的，那么在安装的时候由系统的包管理器将这些广播注册上去
* 接收到广播之后，这个BroadcastReceiver对象是有效的，当onReceive方式完成之后，这个BroadcastReceiver对象被视为无效的
* 广播对进程状态的影响，如果onReceive正在执行，那么这个进程被认为是前台进程
发送广播
* sendOrderedBroadcast
* sendBroadcast
* LocalBroadcastManager.sendBroadcast：在app内发送广播消息
虽然Intent可以发送广播或者启动activity，但是他们之间是不可见的；也就是说BroadcastReceiver不能看到启动activity的Intent，用来启动BroadcastReceiver的intent也不能启动activity
安全和好的实践
如果广播只在app内使用，那么请使用局部广播
最好动态注册广播
发送广播时指定广播接收者所需权限，设置接收包名
注册广播的时候，可以指定所需的权限，设置android:exported 为false

## 管理设备活跃状态
### 1.设备保持活跃状态
1）在使用wakelock之前，请考虑是不是符合如下情景
* 如果是下载，请使用DownloadManager
* 如果是和外部服务器同步数据，请使用sync adapter
* 如果是一个后台服务，请使用jobScheduler以及Firebase cloud message
2）保证屏幕打开
* getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_
SCREEN_ON)
* android:keepScreenOn="true"
通过这个flag让屏幕保持打开状态，不需要任何权限
3）保证CPU运行
* 第一申请WAKE_LOCK权限，“android.permission.WAKE_LOCK”
* 通过PowerManager来获取WakeLock
使用WakefulBroadcastReceiver来保持设备活跃
### 2.定制循环闹钟
1）闹钟有这些特征
* 在规定的时间或者间隔产生intents
* 你可以将它和广播联合起来使用，通过广播来启动一个服务
* 闹钟是在应用之外启动的，通过闹钟可以在你的应用还没有启动的时候去产生一个事件或者一个动作
* 闹钟可以减少资源使用，你不用使用定时器或者一个长时间运行的后台服务

在打盹模式下，闹钟是不会自动触发的；只有在设备退出打盹模式之后才会触发；
有两种方法可以保证你的任务可以执行完成：
* AlarmManager 的setAndAllowWhileIdle
* 使用WorkMananger
最佳实践
* 在循环闹钟进行网络请求时，请尽量将请求散列开来，这样可以减小服务器的负担
* 循环频率尽可能小
* 不要使用精确的时钟触发机制，使用setInexactRepeating()代替setRepeating()
* 不要使用精确的时间点来触发闹钟

## 应用数据和文件
学习如何将你的应用和用户数据保存在设备里，你可以把它保存为文件，健值对，数据库或者其他类型；并且可以将数据和其他应用或者手机共享；或者你可以通过后台服务将用户数据保存在云端，通过云可以将数据和其他设备同步

### 1.数据和文件存储
* 内部文件存储：应用私有
* 外部文件存储：公有
* Share Preference：应用私有
* 数据库：应用私有
如果你想和其他应用共享文件，请使用FileProvider
如果你想提供数据给其他应用，请使用ContentProvider
1）内部存储
文件系统为每个应用提供一个私有的目录，当删除应用时，保存在内部存储的文件就会被删除。
内部缓存文件：如果你想临时保存文件，那么就使用缓存目录；当内部存储空间不足的时候，系统会先删除缓存文件；在应用删除时，这个内部缓存文件也会被删除
2）外部存储
每个安卓设备都有一个外部存储空间用来保存文件。但是外部存储不保证一定可以被访问，它可以被物理的移除（如sd卡），所以在访问外部存储之前，一定要判断外部存储的状态
3）Shared preferences
可以保存int，long，boolean，string，float，这些健值对被写入XML文件，你可以指定这个xml文件的名称
4）数据库
优先使用Room这个库来操作数据库
安卓sdk提供sqlite3数据库工具来显示数据表内容，包括执行sql语句等

### 2.保存文件在设备存储器
因为文件保存路径在不同的设备中有可能不同，所有最好使用系统提供的接口去访问文件，最好不要用绝对路径
1）选择内部或者外部存储
外部存储可能是sd卡，也可能是将一个存储设备划分成两块，一块是内部存储，一块是外部存储。不过在使用相关api的时候，不需要关注这个外部存储是否是可以移除的
2）内部存储器
* 它一直是可用的
* 保存在这的文件只有本应用可以访问
* 当应用删除时，这些文件将被删除
3）外部存储
* 它不一定是可用的
* 它是全局可以访问的
* 当用户删除应用时，如果你是用getExternalFilesDir访问来创建文件的，那么这些文件会被删除，否则，不会被删除。
注意默认情况下应用是被安装在内部存储器中的，不过你可以在manifest中设置 android:installLocation属性，从而可以将应用安装在外部存储中。
4）保存文件在内部存储中
getFilesDir()
getCacheDir()：在读取之前需要确认文件是否存在
getDir(name, mode)
openFileOutput(filename, Context.MODE_PRIVATE)
openFileInput(name)

5）保存文件在外部存储
在访问外部存储之前，先申请外部存储访问权限，然后再验证外部存储是否可用（可能被加载到pc上或者把sd卡移除）
getExternalFilesDir()：在android4.4（api19）之后不需要申请外部访问权限
1）保存文件在公共目录
 Environment.getExternalStoragePublicDirectory()
2）保存文件在私有目录，删除应用时，这个目录被删除
getExternalFilesDir()

### 3.保存健值对
1）获取shared preference的引用
* getSharedPreferences()：通过context都可以获取SharedPreferences，并且可以通过名字来区分不同的SharedPreference
* getPreferences()：只有activity才有这个接口，获取一个activity对应的默认SharedPreference
* getDefaultSharedPreferences：获取整个应用的默认SharedPreference

2）写入SharedPreference
* apply：立即写入内存，不过异步更新在磁盘中
* commit：同步写入磁盘，不要在UI线程做这个操作

### 4.使用Room将数据保存在数据库中
在Room中有三个主要的组件：
* Database：主要负责和应用的持久层的关系数据相连接
* Entity：代表数据库中的一个数据表
* DAO：包含所有访问数据库的方法
### 1.使用DAOs来访问数据
DAO可以是一个接口或者抽象类，如果是抽象类，那么构造函数有一个参数，参数是RoomDatabase
1）定义访问数据库的方法
insert
query
update
delete
2）查询信息
基础查询
根据条件查询
返回部分列
根据集合参数查询
被观察者查询
响应式查询
直接cursor访问
3）Room数据库迁移
4）Room使用复杂数据类型
使用TypeConverter注解

### 5.保存简单的数据
在应用之间传递简单数据，使用Intent或者ActionProvider
发送简单数据到另一个应用
* 发送文本：Intent.ACTION_SEND，Intent.EXTRA_TEXT
* 发送二进制内容：使用Uri代表一个二进制内容：Intent.ACTION_SEND，Intent.EXTRA_STREAM
* 发送多个内容：Intent.ACTION_SEND_MULTIPLE，Intent.EXTRA_STREAM
从其他应用接收简单数据

### 6.共享文件
最安全的做法就是将你应用需要共享的文件的Uri共享给其他应用，并且给这个Uri一个临时授权。FileProvider提供一个方法getUriForFile来生成一个文件内容Uri
建立文件共享
* 在Manifest文件中定义provider
* 在xml文件中定义需要共享的文件目录
共享文件

### 7.使用NFC共享文件
### 8.打印文件
### 9.content providers
ContentProvider帮助应用去管理数据访问，包括访问应用本身的或者其他应用的数据，提供了一种可供其他应用访问的共享数据。
依赖于ContentProvider的类
* AbstractThreadedSyncAdapter
* CursorAdapter
* CursorLoader
如果你想用如上的类，那么你就需要实现ContentProvider。
安卓系统本身提供了一些content provider，包括音频，视频，图片，个人联系信息等
1）ContentProvider基础
provider对外提供一个或者多个数据表
一般来说，通过UI访问provider都是通过CursorLoader来在后台执行异步操作。
Content Uri
* 一个content uri是一个uri，它是用来识别在provider中的数据的。content uri包括整个provider的名字（authority）以及一个名字，这个名字用来指向一个数据表（path）
* 好多provider允许访问一个数据表中的一行，通常都是通过在Uri的后面加上一个id来实现的
* Uri以及Uri.Builder类包含了好多非常方便的方法，通过这些方法能够从字符串创建格式化好的Uri。ContentUris类提供方便的方法添加id到Uri中
Content provider权限
如果一个provider应用没有指定权限，那么其他应用不能访问这个provider，不过这个provider应用内的其他组件可以随便访问这个provider。
provider数据类型
* integer, long, float,double,blob
* 每列的数据类型，cursor.getType()
*  每个content Uri也可以有Mime type，通过ContentResolver.getType()来获取
通过其他几种形式来访问provider
* 通过batch访问，ContentProviderOperation类，可以同时操作几个数据表
* 使用CursorLoader异步访问
* 通过intent来访问provider
2）创建content provider
什么时候需要创建content provider？
* 如果你想给其他应用提供复杂的数据或者文件
* 如果你允许用户从你的应用获取复杂的数据
* 如果你想给查询框架提供自定义的查询建议
* 如果你想将你的应用数据暴露给widgets
* 如果你想实现AbstractThreadedSyncAdapter, CursorAdapter, 或者CursorLoader类
如何创建content provider？
* 原始的存储类型，包括文件数据以及结构化数据（类似数据库表单）
* 实现ContentProvider类
* 定义provider的autority字符串，它的Uris以及列名
* 添加一些其他实现，如sample数据或者实现AbstractThreadedSyncAdapter，通过它可以将provider和云进行数据同步
数据存储
* 如果是结构化数据，那么就使用sqlite数据库，如果是文件，比如图片，音频，视频等，那么就使用文件
* 安卓可以使用Room来操作数据库
* 只有小部分的情况下，你需要在一个应用中实现多个provider。比如说使用一个provider给widget共享数据。另一个provider共享给其他应用
数据设计需要考虑的因素如下：
* 每个表需要一个主键，使用_ID表示
* 如果需要提供一个bitmap或者大文件，那么先保存这个文件，并且间接的提供该文件，如使用FileProvider
* 使用二进制数据类型来保存数据，比如协议缓存或者json文件
设计Content Uris
* authority代表整个content provider，path代表一个数据表，id代表一个表的行
* 每个数据访问方法都有一个uri作为参数，用来指定数据表，行或者文件
::请使用Uri相关的类，Uri，Uri.builder，ContentUris以及UriMatcher::
实现ContentProvider类
实现query，insert，update，delete，getType以及onCreate，除了onCreate，contentResolver有对应的接口
::需要注意的是，除了onCreate以外，其他接口都需要保证线程安全::
::避免在onCreate做一个耗时操作::
实现ContentProvider MIME类型
实现Contract类
实现ContentProvider权限

3）实现SAF来打开文件
安卓4.3之前，如果你想从其他应用获取一个文件，那么请使用ACTION_PICK或者ACTION_GET_CONTENT。
安卓4.4之后，你可以使用ACTION_OPEN_DOCUMENT，通过这个系统UI，你能够从所有支持的应用中选择一个文件
* 不同点：
ACTION_GET_CONTENT可以获取一个数据拷贝，而ACTION_OPEN_DOCUMENT则是通过UI可以对相应的文档进行编辑，是直接操作content provider的
可用操作
打开文档
创建文档
删除文档
编辑文档
4）创建一个自定义的document provider
ACTION_GET_CONTENT在android 4.4之前使用
ACTION_OPEN_DOCUMENT在android4.4之后使用，并且这两者不要同时使用，要不然会导致不同的用户体验
* 在manifest文件中定义这个provider
* 定义Contracts类
* 创建DocumentsProvider的子类
你需要覆盖如下方法
queryRoots()
queryChildDocuments()
queryDocument()
openDocument()
### 10.应用安装
从Api8开始，允许应用安装在外部存储中
android:installLocation="preferExternal”，还有auto，internalOnly
应用被安装在外部存储时
应用的性能没什么影响
.apk保存在外部存储中，但是私有数据，databases，优化的.dex文件以及解压的代码都放在内部存储中

## 用户数据和身份
### 1.登陆流程
### 2.自动输入框架
### 3.日历provider
* 你可以对日历的日程，事件，参与者，提醒等进行增删改差操作
* 应用或者sync adapter可以使用Calendar provider的api
* 为了操作日程数据，需要申请相关的权限
* 为了更加方便的操作日程，Calendar provider提供了一些intents来对日程进行增加，查看以及修改
* 用户和日程应用进行交互，然后把数据返回给请求数据的应用
* 每个provider提供一个公开的Uri来识别这个数据集
* 每个provider的Uri都是以content://开头的，依此来表明这个数据是来自provider的

日历的数据表包括
CalendarContract.Calendars：每一行代表一个日程
CalendarContract.Events：每行代表一个事件的详细信息
CalendarContract.Instances：每行代表一个事件的起始和结束时间
CalendarContract.Attendees：每行代表一个参与者
CalendarContract.Reminders：每行代表一个提醒
Calendars表
* 如果你需要查询Calendars.ACCOUNT_NAME，则需要把Calendars.ACCOUNT_TYPE也一起带上，这是因为一个账号是否唯一需要考虑ACCOUNT_NAME和ACCOUNT_TYPE
* 如果你需要插入一个calendar，那就使用sync adapter
Events表
Attendees表
Reminders表
Instances表
这个表不允许写入？只有增加events的时候，由写events写入？
Sync adapters
通过sync adapters能够操作更多的数据表列

### 4.联系人provider
### 1.Contacts Provider的概览
可以通过应用程序去访问联系人provider，或者将联系人provider同步到服务器去
* ContactsContract.Contacts表：每行代表不同的人，它是通过聚合raw contact行来实现的
* ContactsContract.RawContacts表：根据用户的账号和类型，包含了一个用户的数据
* ContactsContract.Data表：包含了用户raw contact的详细数据
Raw contacts
* 大部分的raw contacts数据不是保存在ContactsContract.RawContacts表中的，而是保存在ContactsContract.Data中，可能占据data中的一行或者多行
* 每个data行包括Data.RAW_CONTACT_ID
* 一个raw contact的名字不是保存在ContactsContract.RawContacts表中的，它是保存在ContactsContract.Data中的，在ContactsContract.CommonDataKinds.StructuredName行中，每个raw contact在ContactsContract.Data行中，只有一行是这种类型的
* 为了保存数据到raw contacts，请先使用accountManager把account name和account type添加进去
Data
* 一个raw contact可能指向多个类型相同的data行
* 多个不同类型的数据保存在一个data数据表中
* 有些列在不同类型的含义是相同的（描述性列名），有些列在不同类型时的含义是不同的（通用的列名）
来自Sync adapter的data
* 联系人信息一部分来自用户输入，还有一部分是通过sync adapters从网络中导入的
* 如何你想你的服务器和联系人provider进行数据传输的话，你需要自己写一个sync adapter
* 一个sync adapter是通过它的account type来标志的
使用者信息user profile
ContactsContract.Contacts表中有一行持有了使用者的用户数据，这个contacts row连接到一个raw contacts row上，raw contacts row可能关联到多个data rows
需要使用READ_PROFILE和WRITE_PROFILE权限
联系人provider元数据
联系人provider访问
访问实体：通过访问实体，可以一次性访问row以及它的child rows，不过实体一般不完全包括父表以及子表的所有列
批量修改：在contacts provider中批量插入，更新以及删除
通过intents获取或者修改联系人信息
数据完整性
自定义数据行
sync adapter
* 自动检查网络是否可用
* 安排和执行同步
* 会自动重新同步
一个sync adapter只能服务于一个服务和content provider
### 2.获取联系人列表
使用CursorLoader来访问数据
通过名字来获取联系人列表
通过数据类型来获取联系人列表
获取任何类型的联系人列表
### 3.获取联系人详情
### 4.使用intents来修改联系人
### 5.显示快速联系人图标

## 用户位置
### 1.优化定位，节省电池
android 8.0对后台定位的限制如下：
* 在后台收集定位信息被掐死，现在的定位信息是通过计算的，并且一个小时只会发送几次
* Wifi定位更为精准，当一个设备一直连在一个静态访问点时，位置信息将不会被计算
* 地理围栏每2分钟触发一次，代替之前的每10秒一次
电池的损耗跟哪些因素相关：
* 精度
* 频率
* 时延
定位使用场景
* 用户可见并且是前台更新，如地图
* 获取设备的地理位置，如天气预报
* 用户进入一个特定位置时开始更新，使用电子围栏
* 在用户处于活跃状态时进行更新，比如在开车或者骑自行车的时候
* 地理位置相关的长时间的后台位置信息更新，如电子围栏
* 没有一个可见的组件并且长时间进行位置更新
* 用户操作其他应用时使用高频高精度的位置信息更新
定位最佳实践
* 在不需要的时候关闭位置信息更新
* 设置超时
* batch请求
* 使用积极的位置信息

### 2.获取最后的位置信息
### 3.更改定位设置
### 4.接收位置更新
### 5.显示位置更新
### 6.显示定位地址
### 7.创建和监控电子围栏

## 传感器
### 1.传感器概述
安卓系统提供三种类型的传感器
* 运动传感器：加速传感器（摇动，倾斜），重力传感器，陀螺仪（测量设备在x，y，z的转动速度）以及旋转向量
* 环境传感器：气压计，光度计以及温度计
* 定位传感器：方向传感器以及磁力计
传感器framework
SensorManager
Sensor
SensorEvent
SensorEventListener
### 3.运动传感器
大多数的安卓设备有加速传感器，现在大部分设备也有陀螺仪。大部分软件传感器是依赖一个或者多个硬件传感器。
### 4.定位传感器
地磁传感器以及距离传感器
### 5.环境传感器
光，压力，气温传感器
## Connectivity连接
除了网络连接以外，android还提供API供应用使用，通过这些API应用可以和其他设备进行蓝牙，NFC，wifi P2P，USB以及SIP传输
### 1.执行网络操作
1）连接到网络
保证安全的网络传输
* 尽可能的减少敏感信息在网络上传输
* 使用ssl在网络上传送信息
* 创建网络安全配置，指定信任的CAs
选择Http的客户端
使用HttpsURLConnection客户端，支持TLS，上传和下载，配置超时，ipv6以及连接池
在一个线程上进行网络操作
所有的网络操作包含在一个fragment中
通过一个fragment来实现整个网络操作，这样可以在下载过程中，如果出现配置变化导致的activity重建。我们可以通过fragmen manager来重新找回这个fragment，重新将这个fragment绑定到新的activity上去，这样可以继续更新这个新的activity。当然在fragment的oncreate中需要调用setRetainInstance(true)
2）管理网络
检查设备的网络连接：ConnectivityManager，查询网络连接以及网络状态变化监控；NetworkInfo：描述某种类型网络状态
管理网络使用：有一个网络管理Activity，并且使用android.intent.action.MANAGE_NETWORK_USAGE
3）优化网络数据
4）解析xml数据
### 2.使用Volley来发送网络数据
1）Volley总览
使用Volley有如下优点
* 自动安排网络请求
* 多连接并发
* 使用磁盘和内存缓存
* 支持请求优先级
* 支持取消请求
* 方便自定义，比如重试和补偿
对于大文件下载或者流操作，Volley并不适用，这是因为Volley在parse的过程中会在内存中保存所有的responses。如果有大文件下载，请使用DownloadManager
2）发送一个简单请求
发送一个请求
Volley中有一个缓存处理线程和网络请求线程池。当你将一个网络请求添加到请求队列之后，首先它被缓存处理线程处理，如果这个请求命中了cache，那么就将缓存的响应进行解析，并且将解析后的响应发送给主线程。如果网络请求没有命中缓存，那么将它放入网络请求队列。当有空闲的线程时，这个线程会从网络请求队列中取出一个网络请求，进行http的交互，并且对解析响应，将响应写入缓存中，并且将解析后的响应发送给主线程。
取消请求
调用request的cancel可以取消一个请求。
对于一个activity，如果你想在onstop取消所有的请求，那么在添加请求的时候给request设置一个tag，然后调用requestQueue的cancelAll就可以了
3）建立一个请求队列
一个请求队列需要两个东西来让它工作：一个网络来处理请求，一个缓存来处理缓存。
在应用中最好使用singleton设计模式来使用RequestQueue
4）建立一个标准的请求：StringRequest，JsonObjectRequest，JsonArrayRequest
5）实现一个自定义的请求
### 3.使用Cronet来执行网络操作
* 支持http，http2，quic
* 支持按优先级进行请求
* 支持内存以及磁盘缓存请求响应
* 异步请求
* 支持数据压缩
### 4.不要在传输数据时过度消耗电池
怎样在下载和网络连接时降低电池的消耗？
1）根据网络状态优化下载
无线状态机
2）如何减少日常的更新
* 基于设备的状态，网络连接，用户行为以及用户的喜好来优化日常更新
* 在网络断链后禁止后台服务更新，在电量较低的时候较少日常更新
* 使用FCM，使用push不是poll
* 使用JobScheduler
3）不要冗余下载，就是要求只下载需要下载的东西
* 下载小图片
* 使用缓存
4）根据连接类型来调整下载模式
Wi-fi的带宽更大，而且使用更少的电池
### 5.减少网络电池消耗
对于一个应用来说，网络请求会导致消耗大量的电量；除了在发送和接收数据包的时候会消耗电量，在打开网络和保持激活状态也会消耗电量；每15秒钟一次网络请求会导致无线网络一直处于激活状态，从而导致电量持续消耗。
1）收集网络数据包
网络请求来源：用户的操作，应用的网络请求，服务器的通信请求
* 给网络请求打上标签：用户，应用或者服务器
* 配置一个网络测试编译类型
* 安装这个网络测试应用
* 使用网络包分析工具
2）分析网络数据包
对于用户的网络请求：
可能需要预先拉取数据
在网络请求前先检查网络状态
每次网络请求拿到一个数据集合
对于应用的网络请求：
可以将多次网络请求放在一起
对于服务器的网络请求
考虑使用GCM
3）优化用户的网络使用
* 预先拉取网络数据
* 检查网络，监听网络变化
* 减少网络连接，尽可能使用现有的连接
4）优化应用网络使用
在移动设备中，打开无线网络，建立连接以及保持无线连接需要大量的电量
* 打包和安排网络请求
使用打包和安排API，如GCM，JobScheduler以及Sync Adapter
* 允许系统去检查连接
使用scheduler，它会自动使用Connectivity Manager去检查连接，如果没有连接，它不会启动应用去连接网络，从而节省电源。
5）优化服务器的网络使用
不要使用pull技术从服务器拉取，而是使用push；使用GCM
6）优化通用的网络使用
* 压缩数据
* 使用本地缓存
* 优化缓存大小
### 6.使用sync adapters来传送数据
使用sync adapters的好处
* 插件化结构
* 自动执行
* 自动的网络检查
* 提高电池性能
* 帐号管理和授权
1）创建一个存根认证器
默认情况下，认为设备和服务器之间同步数据需要一个账号，所以需要一个认证器。即使你的应用不需要使用账号，那也需要提供一个认证器。除此之外，你还需要绑定一个服务，这个服务供sync adater来调用认证方法。
* 创建一个存根认证器
* 创建一个service，并且返回这个存根认证器的代理
* 添加一个认证器元数据文件：为了将你的认证器组件集成在sync adapter以及account框架中，你需要给这些框架提供元数据。
* 在manifest中定义这个认证器
sync adapter除了需要一个认证器外，还需要一个存根Content provider。
2）创建一个存根Content provider
设计sync adapter框架目的是为了和Content provider一起配合工作。如果你已经保存了本地数据，但不是在Content provider中，那么就添加一个存根Content provider。
* 添加一个存根Content provider
* 在manifest中定义这个存根Content Provider
3）创建一个sync adapter
这个组件包括了设备和服务器进行数据同步任务的代码。sync adapter框架会自动的执行这个sync adapter组件中的代码。
一个sync adapter组件包含如下部分：
* 一个Sync adapter类
* 一个绑定服务
* sync adapter的xml元数据文件
* 在Manifest中定义绑定服务以及元数据文件
4）运行一个sync adapter
什么时候运行sync adapter，需要避免通过用户操作来触发sync adapter。
* 服务器数据变化的时候
* 设备数据变化的时候
* 一个日常的间隔
* 按需
### 7.Bluetooth
### 8.NFC
### 9.Telecom，可以建立一个打电话的app
### 10.USB

## 点击和输入
包括触摸以及手势输入，键盘以及游戏控制
### 1.输入事件概览
1）事件监听器
* onClick()
* onLongClick()：有一个返回值，如果true表明你已经处理了这个事件，事件处理到此为止
* onFocusChange()
* onKey()：有一个返回值，如果true表明你已经处理了这个事件，事件处理到此为止
* onTouch()：有一个返回值，如果down事件返回一个false，那么表明你没有消费这个事件，同时也表明你对之后的事件序列也不感兴趣。
* onCreateContextMenu()
2）事件处理者
Activity.dispatchTouchEvent(MotionEvent)：允许activity拦截所有的事件
ViewGroup.onInterceptTouchEvent(MotionEvent)：允许ViewGroup监听所有发给子View的事件
ViewParent.requestDisallowInterceptTouchEvent(boolean)：让ViewGroup不要拦截事件
3）接触模式
isFocusableInTouchMode（）：在接触模式下可以被选中，如EditText。但是在使用轨迹球或者键盘时，button等组件也可以被选中
接触模式状态是被系统维护的，你可以通过isInTouchMode() 来了解一个设备是否处于接触模式。
4）处理选中
### 2.使用手势
1）通用手势识别
怎么做手势识别？
* 收集接触事件
* 根据收集到的事件数据看看是否符合手势的标准
支持库类：GestureDetectorCompat和MotionEventCompat（注意，它不是MotionEvent的替代品）
1.1）收集手势数据
获取Activity或者View的接触事件
* 通过复写Activity或者view的onTouchEvent，然后通过事件的类型可以做自定义手势处理。如果不需要自定义手势，可以直接使用GestureDetectorCompat或者GestureDetector来识别双击，长按，滑行等手势
1.2）检测手势
* 在初始化GestureDetectorCompat的时候，有一个参数就是GestureDetector.OnGestureListener。
* 如果你只想处理部分手势，那么使用GestureDetector.SimpleOnGestureListener。不过onDown一般都需要返回true，因为每个手势都是由down事件开始的，除非你真的想忽略这个手势。
### 3.查看接触和指针移动
由于手指的操作一般不那么精确，所以检测接触事件一般是基于多个移动事件的，而不是基于单个的操作。
有好几种方法来检测手势中的移动
* Pointer的开始和结束位置
* Pointer移动的方向
* 通过事件历史
* Pointer的加速度
3.1）追踪加速度
使用VelocityTracker
VelocityTracker.obtain()
VelocityTracker.addMovement(event)
VelocityTracker.computeCurrentVelocity(1000)
请在ACTION_MOVE计算加速度，不要在ACTION_UP计算，不然计算结果会为0
3.2）获取pointer
请求一个View来获取鼠标指针，如果获取成功，那么后续的鼠标事件将会被发送到该View。
### 4.让滑动手势执行动画
你可以使用Scroller或者OverScroller来进行一个滑动动画。scroller实际上不会直接作用到View上。它本身只会计算滑动的offset，不会将这些offset作用到View上去。具体如何作用到View上是你的工作，不是scroller的工作。
4.1）理解滑动术语
拖拽和滚动
4.2）实现滑动
### 5.实现多点触碰手势
ACTION_DOWN：第一个碰到屏幕的手指，这是一个手势的开始，一般都是索引0
ACTION_POINTER_DOWN：除了第一个手指外的手指碰到屏幕时会产生，可以通过MotionEvent的getActionIndex获取到它的索引值
ACTION_MOVE
ACTION_POINTER_UP：非首要手指离开屏幕
ACTION_UP：最后一个手指离开屏幕
你可以通过每个手指的索引和id来跟踪一个指定的pointer。
Index：一个pointer的index是它在这个数组的位置，大部分的MotionEvent的方法都是使用index作为参数的，而不是pointer的ID
ID：每个pointer都有一个id，这个pointer的id在一个手势里保持不变。通过这个ID可以在整个手势中跟踪一个pointer。
### 6.拖拽和缩放
### 7.管理ViewGroup的触摸事件
* 当在ViewGroup表面（包括它所包含的子View）触发触摸事件时，onInterceptTouchEvent方法会被调用。如果它返回true，则表明这个MotionEvent被拦截了，MotionEvent将不会被发送给子View。
* onInterceptTouchEvent给了ViewGroup一个机会，在把MotionEvent发送给子View之前可以观察这个事件。如果onInterceptTouchEvent返回true，那么之前接受MotionEvent的子View将会接收到一个ACTION_CANCEL。此后，MotionEvent将会被发送给ViewGroup的onTouchEvent进行处理。
7.1）拦截事件
7.2）使用ViewConfiguration常量
7.3）扩展一个View的接触面积
### 8.处理键盘输入
Android支持软键盘，也支持硬件键盘
8.1）指定输入方法类型
* 在EditText中指定android:inputType
* 打开拼写建议功能android:inputType=
        "textCapSentences|textAutoCorrect"
* 指定输入法的动作android:imeOptions="actionSend”，TextView.OnEditorActionListener
* 提供自动完成建议
8.2）显示和隐藏输入法界面
* 在activity启动的时候显示输入法界面，需要在manifest中指定<activity android:windowSoftInputMode="stateVisible"
* 按需显示输入法界面使用InputMethodManager，imm.showSoftInput(view, InputMethodManager.SHOW_IMPLICIT)
* 指定UI的响应方式
8.3）支持键盘导航
Android设备支持触摸导航，也支持键盘导航
* 处理Tab导航
* 处理方向导航
8.4）处理键盘actions
默认情况下，系统已经帮我们处理好键盘输入。不过如果想自己拦截或者处理键盘输入，那么请实现KeyEvent.Callback。
* 处理单个键事件
* 处理modifier键
### 9.支持游戏控件
让玩游戏的人通过他们喜欢的游戏控制器来玩游戏以提高游戏体验
9.1）处理游戏控制器动作
安卓系统会将游戏控制器的事件码转换为安卓的键盘码或坐标。通过这些键盘码和值转化为游戏的动作。一般是通过KeyEvent以及MotionEvent来检测游戏控制器的按键
* 确认游戏控制器是否已经连接
* 处理gamepad按键事件
* 处理pad输入
* 处理游戏摇杆移动
9.2）游戏控制器支持多个安卓版本
9.3）支持多种游戏控制器
### 10.输入法
为了给安卓系统增加一个输入法，创建一个类来继承InputMethodService。
需要一个设置界面来将输入法选项配置给InputMethodService
10.1）创建一个输入法
在Manifest文件中申明输入法，包括service和设置界面
输入法API：KeyboardView，BaseInputConnection，InputMethodService
10.2）图形键盘
10.3）设计输入UI
10.4）将文字发送到应用中
10.5）创建一个输入法子类型
10.6）切换输入法子类型
10.7）创建输入法建议
### 11.拼写检查
安卓框架提供拼写检查框架，你可以实现拼写检查框架，并且使用该检查框架
11.1）实现拼写检查服务SpellCheckerService
11.2）访问拼写检查服务
## Web相关内容
### 1.概览
通过webview你可以控制web显示的大小或样式。
你可以在原生应用中定义接口，然后在web中通过javascript调用。
### 2.使用WebView来建立web应用
* WebView是View的子类，它没有实现浏览器的所有功能，如导航，地址栏等，它默认只能浏览页面
* 如果信息可能会更新，如用户手册或者用户指南等，那么使用WebView。
2.1）把WebView添加到布局文件中
* 在布局中添加一个WebView
* 在Manifest中添加网络访问权限
如果你想自定义WebView，那么请注意
* 通过WebSettings来启动javascript
* 使用javascript来访问安卓接口
* 使用WebViewClient来拦截Url加载，处理影响页面内容展示的事件，如表单提交出错，导航等
* 使用WebChromeClient来支持全屏展示，创建或者关闭窗口，展示提示框等。
2.2）在WebView中使用javascript
* 如果在WebView中展示的页面使用了javascript，那么请打开javascript开关
* javascript开关打开之后，你也可以创建原生和javascript之间的接口
* 使用@JavascriptInterface注解来表明这个方法提供给javascript使用
* webView.addJavascriptInterface(WebAppInterface(this), "Android")
* 需要注意的是绑定到javascript的对象是运行在其他线程中，不是实际创建这个对象的线程
2.3）处理页面导航
默认情况下，如果你点击一个webview中的url，它会启动浏览器去加载这个页面链接。
如果想在WebView中打开这个url，那么就设置一个WebViewClient。
### 3.管理WebView对象
3.1）获取版本号的API
3.2）谷歌安全浏览API
### 4.将WebView迁移到安卓4.4
4.1）User agent改变
4.2）多线程处理
4.3）自定义URL处理
shouldOverrideUrlLoading()或者shouldInterceptRequest()只在合法的URL才会调用
4.4）Viewport改变
4.5）样式的改变
4.6）在javascript处理接触事件
### 5.支持不同屏幕
需要考虑如下两个主要因素
Viewport：一个矩形用来绘制web页，包括大小和缩放
屏幕密度：
5.1）指定Viewport属性
5.2）可以根据手机密度来指定css
5.3）使用javascript来指定设备密度
### 6.调试Web App
6.1）在安卓浏览器中运行Console API：javascript中调用console.log
6.2）在webview中调用Console API
### 7.Web应用的最佳实践
7.1）使用为mobile量身定做的链接，一般是使用服务器端重定向。一般是通过浏览器发送的User agent字符串来识别的。
7.2）使用HTML5 DOCTYPE
7.3）使用viewport元数据来改变Web的大小
7.4）使用竖向浏览
7.5）在使用Webview时，需要设置为match_parent
7.6）避免多文件请求

如何使用Webview？
* 在布局文件中定位WebView，并且宽高都需要设置为match_parent，并且包括这个Webview的父控件也需要设置为match_parent。
* 在manifest文件中开启网络使用权限
* 如果Webview加载的html使用javascript，那么请启用javascript
* 如果用户点击Webview里面的链接，默认会使用浏览器打开这个链接。如果需要使用Webview打开这个链接，那么需要设置WebViewClient。
* 如果想重写url，那么请重写shouldOverrideUrlLoading()，并且返回true
[Android：这是一份全面 & 详细的Webview使用攻略 - 简书](https://www.jianshu.com/p/3c94ae673e2a)

## MVC、MVP和MVVM
[Android App的设计架构：MVC, MVP, MVVM - 简书](https://www.jianshu.com/p/3010760035e0)
[Android App的设计架构：MVC,MVP,MVVM与架构经验谈](https://zhuanlan.zhihu.com/p/20852740)
[Android App的设计架构：MVC,MVP,MVVM与架构经验谈-android,mvp,mvc 相关文章-天码营](https://www.tianmaying.com/tutorial/AndroidMVC)
## Dagger2
[Dagger2从入门到放弃再到恍然大悟 - 简书](https://www.jianshu.com/p/39d1df6c877d)
### 1.依赖注入方式
* 通过接口注入
* 通过set方法注入
* 通过构造函数注入
* 通过java注解注入
### 2.提供依赖的方式
* 在一个构造函数里使用@Inject注解，通过构造方法生成对象来提供依赖
* 在一个Module类中使用@Providers标注方法，通过这个方法提供依赖
### 3.Dagger2注入原理
通过apt插件在编译阶段生成相应的注入代码
所有的依赖都是通过factory提供的
[详解 Dagger2 系列，原来 Dagger2 如此简单 - Android - 掘金](https://juejin.im/entry/578cf2612e958a00543c45a4)
@Inject：一个是用在构造函数上，一个是用在标记一个变量。通过构造函数提供一个依赖。标记一个变量表明它需要通过依赖注入。
@provide：provider标记一个Module里的方法，这个方法在需要提供依赖的时候被调用。
@Module：用module标志的类是专门用来提供依赖的，因为有一些类不能给构造函数使用@Inject，如引用的库等（另一种是通过在构造函数中使用@Inject来提供依赖，优先使用@Module里提供的依赖）
@Component：用来标志接口，被标注了Component的接口在编译的时候会产生相应的类实例

Dagger2工作原理：
* 从使用@Inject修饰的需要注入的变量开始的，然后从module或者@Inject注解过的构造函数获得实例，然后通过Component标志的接口将获得的实例注入到需要依赖的变量中。
* module中定义的实例优先构造函数生成的实例。
* 构造实例过程中所需的参数也是先从module中找，如果不存在，再从构造方法中找。 

## RxJava
[RxJava 沉思录（一）：你认为 RxJava 真的好用吗？ - 掘金](https://juejin.im/post/5b8f536c5188255c352d3528)
[给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083#toc_31)
## 屏幕适配
[Android屏幕适配全攻略(最权威的官方适配指导) - CSDN博客](https://blog.csdn.net/zhaokaiqiang1992/article/details/45419023#android%E5%B1%8F%E5%B9%95%E9%80%82%E9%85%8D%E5%87%BA%E7%8E%B0%E7%9A%84%E5%8E%9F%E5%9B%A0)

## JScript
[Android JSBridge的原理与实现 - CSDN博客](https://blog.csdn.net/sbsujjbcy/article/details/50752595)

## 基本知识
[Android 知识梳理 - 掘金](https://juejin.im/post/587dbaf9570c3522010e400e)

## 系统架构
1.通过dagger2将Map<componentName,Class<BaseComponentImpl>>,
Map<componentName, Class<DataProvider>>,
MenuProvider以及SettingProvider注入到系统中
2.通过注入的这些参数来初始化AppKit，其中AppKit包括NavigatorImpl，LogManager，SettingProvider以及MenuPovider，并且初始化ComponentFactoryImpl
### 1.启动一个component
3.如果需要启动一个component，那么此时需要通过BaseComponnetAcitivyt的startComponent方法来启动，其实质是通过调用
getAppKit().getNavigator().open( activity, uri, requestCode )来启动component的
4.通过调用NavigatorImpl的open( Activity activity, String uri, int requestCode )方法来调用。
5.在open方法中会调用ComponentFactoryImpl中的create方法来创建component。创建时会使用从dagger中获取的component的class对象来创建这个component对象
6.在NavigatorImpl中会维护一个component启动的栈。这个时候会设置各个component的状态，包括caller以及是否处于活跃状态等。
7.最终component会调用component.start( Activity activity, int requestCode )
来启动这个component。
8.启动activity，其实质是通过记录在component的componentId，componentName，以及使用的Activity的class对象等参数来启动的。
### 2.关闭一个component
9.后退的时候关闭一个Activity，会在BaseComponentActivity的finish()方法中通过getAppKit().getNavigator().popup( activity, mComponentId )来将这个component退出，也就是从navigator的调用栈中退出
### 3.log系统
在BaseApplication通过是不是Debug模式来设置LogrImpl。如果是release版本，那么只记录错误日志；如果是debug版本，那么记录所有日志。
### 4.performance日志
使用AppKit初始化的LogManager来实现
* LoggerHashmap：通过componentName为健值保存Logger，这样可以为相同component只提供一个Logger（因为同名的component可能会启动多个）。
* LogConfigurationHashmap：通过logConfiguration为健值，保存subscription，这样每个logConfiguration只会有一个subscription。
* LogEventPublishSubject：通过这个subject可以发送onNext的logEvent到subscription中，然后通过LogConfiguration的logFilter以及logAppender来处理这个日志。
* 使用Logger.debug等方法来记录日志
### 5.DataProvider
DataProviderManager在Appkit初始化的时候进行了DataProviderManager的初始化
* DataProviderManager包含了以componentName为健值的DataProvider的class对象。
* DataProviderManager包含了以ComponentId为健值的DataProvider实例对象。（为什么不用componentName为健值？因为同一个componentName可能启动多个实例，所以需要通过ComponentId来区分）
* DataProviderManager提供了获取DataProvider实例的接口，提供了释放DataProvider实例的接口
### 6.在React Native中使用DataProvider
* 通过invoke方法来调用DataProvider使用@path注解的接口，并且返回数据
* 使用on方法可以监听Native调用了publish方法（具体实现请参考Instructor的Dataprovider native实现中）

## BbFragment
创建presenter
在onResume和onPause中记录userVisibleHint
创建了Dialog和ExceptionHandling
创建了loadingHelper
## ComponentFragment extends BbFragment
获取componentId和componentName
attach上activity后，将activity转化成FragmentInteractionListener
获取DataProvider
## BbActivity
创建presenter
创建Dialog
创建ExceptionHandling
创建loadingHelper

## BaseComponentActivity
获取componentId，componentName以及Mode
设置启动和关闭activity的动画
获取Appkit
获取DataProvider


## 