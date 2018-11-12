# OkHttp
[Droidcon Montreal Jake Wharton - A Few Ok Libraries - YouTube](https://www.youtube.com/watch?v=WvyScM_S88c&feature=youtu.be)
## OkHttp的使用
### OkHttp进行get请求
* 获取OkHttpClient：OkHttpClient client = new OkHttpClient();
* 构造request对象：
Request request = new Request.Builder()
                .get()
                .url("https:www.baidu.com")
                .build();
将请求封装为Call
* Call call = client.newCall(request);
* 根据需要调用同步或者异步请求方法
* 同步调用,返回Response,会抛出IO异常
Response response = call.execute();
call.enqueue(）
### OkHttp进行Post请求
* 获取OkHttpClient对象：OkHttpClient client = new OkHttpClient();
* 构建FormBody，传入参数：
FormBody formBody = new FormBody.Builder()
                .add("username", "admin")
                .add("password", "admin")
                .build();
* 构建request对象
final Request request = new Request.Builder()
                .url("http://www.jianshu.com/")
                .post(formBody)
                .build();
* 将request封装为call
Call call = client.newCall(request);
* 使用同步或者异步的方式来调用网络请求
call.enqueue()
Response response = call.execute();

## OkHttp的源码分析
[拆轮子系列：拆 OkHttp - Piasy的博客 | Piasy Blog](https://blog.piasy.com/2016/07/11/Understand-OkHttp/)

![](OkHttp/9E65B84C-A087-4D70-B090-1E8259F676A0.png)

通过OkHttp来进行网络请求，需要如下几个步骤：
1. 构建OkHttpClient
2. 构建Request，通过构建报文头，报文体，请求方法，请求地址以及协议版本
3. 通过OkHttpClient这个callFactory来生成一个call
4. 同步请求时，直接执行call.execute()方法；异步请求时，直接将call放在线程池中之行。

### 网络同步请求
#### 1. RealCall的execute方法
先看RealCall的execute()方法，这个方法直接返回Response，主要工作如下：
* 如果这个call已经被执行，直接抛异常
* 将当前需要运行的RealCall加入到Dispatcher的runningSyncCalls队列中
* 调用getResponseWithInterceptorChain获得结果
* 然后将这个RealCall从runningSyncCalls中删除

#### getResponseWithInterceptorChain

![](OkHttp/E07C4976-3FF9-4B9F-A8F8-D80EE87ED038.png)

##### 1. 将所有的拦截器都存放在interceptors列表中
* 配置OkHttpClient时设置的interceptors
* 负责失败重试以及重定向的retryAndFollowUpInterceptor
* 负责把用户构造的请求转换为发送给服务器的请求，把服务器返回的响应转化为用户友好的响应BridgeInterceptor
* 负责读取缓存直接返回，更新缓存的CacheInterceptor
* 负责和服务器建立连接的ConnectInterceptor
* 负责向服务器发送请求数据，从服务器读取响应数据的CallServerInterceptor
##### 2. 生成一个RealInterceptorChain对象，然后调用chain.proceed(Request request)

##### 3. RealInterceptorChain的proceed(Request request, StreamAllocation，streamAllocation, HttpCodec httpCodec, RealConnection connection)
在process方法里主要做了如下处理
1. 取出当前使用的interceptor
2. 生成一个新的RealInterceptorChain对象chain
3. 然后通过然后调用interceptor.intercept(chain)来执行这个拦截器

##### 3.ConnectInterceptor的intercept(chain)
1. 生成一个HttpCodec
2. 然后调用chain.proceed(request, streamAllocation, httpCodec, connection);
##### 4.CallServerInterceptor.intercept(chain)
1. 通过httpCodeC将网络请求发送给服务器，包括网络请求的请求行，请求头，请求参数。
2. 然后通过HttpCodec来读取响应头和httpCodec获取响应体
3. 返回这个Response


