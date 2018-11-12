## Picasso 源码分析
[图片加载框架Picasso － 源码分析 - 掘金](https://juejin.im/post/58b7f59a44d904006acbbb83)
实际上Picasso以及其他图片加载框架的流程类似，一般都会包括如下几步：
1. 构建请求
2. 查看内存缓存中是否存在符合这个请求的图片，如果有则直接回调并显示，否则步骤3
3. 查看磁盘缓存中是否存在符合这个请求的图片，如果有则直接回调并显示，否则步骤4
4. 通过网络请求下载该图片，经过图片后处理，然后缓存到磁盘和内存中

### 1.构建请求
#### 1.1 构造Picasso实例
首先通过Picasso.get()获取一个Picasso实例，当然构造Picasso实例是通过构造器模式来完成的。Picasso的实例构造会初始化如下主要变量：
1. downloader：顾名思义，就是用来下载图片的，默认使用OkHttp3Downloader
2. cache：内存缓存，默认使用LruCache
3. service：下载使用的线程池，默认使用PicassoExecutorService，其实质就是ThreadPoolExecutor，线程池的大小默认为3.
4. dispatcher：分发器，默认使用Dispatcher，将请求分发出去执行。
#### 1.2  构造RequestCreater实例
通过Picasso的load方法生成RequestCreater实例，顾名思义，这个实例是用来生成request的。
#### 1.3  调用RequestCreater的into()方法
* 首先如果使用了fit属性，那么生成一个deferRequestCreator
* 如果使用了内存缓存策略并且存在内存缓存，那么直接将图片显示在ImageView上
* 建立一个ImageViewAction，并且通过picasso提交这个action
#### 1.4 辗转反侧之后，最终会通过Dispatcher的performSubmit(Action action, boolean dismissFailed)将这个Action分发出去
* 如果存在一个BitmapHanter，那么直接attach上这个action就返回
* 如果Executor已经关闭了，那么直接返回
* 建立一个BitmapHunter
* 通过Executor将这个BitmapHunter提交出去，实际上会将这个BitmapHunter生成PicassoFutureTask，然后由Executor执行
#### 1.5 BitmapHunter 
首先会调用hunt()会返回一个bitmap，在这里面做了如下事情
* 如果使用了内存策略并且存在内存缓存，直接将这个图片返回
* 如果不存在内存缓存，那么使用requestHandler.load(Request request, int networkPolicy)去请求图片。这里的requestHanlder分为好几种，主要是根据图片请求到底是从网络，provider，文件，资源id等来调用不同的requestHandler的load方法来获得图片
* 调用transformResult(Request data, Bitmap result, int exifOrientation)
来对图片进行后处理，然后返回这个图片
* 获取到图片后会调用dispatcher的dispatchComplete(BitmapHunter hunter)
#### 1.6 Dispatcher
调用performComplete，在这里做了内存的缓存更新，然后最终辗转反侧会调用ImageViewAction的complete(Bitmap result, Picasso.LoadedFrom from)
方法，在这里将图片设置上去。








