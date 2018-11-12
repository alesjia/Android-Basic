## Retrofit的使用
[Retrofit分析-漂亮的解耦套路 - 简书](https://www.jianshu.com/p/45cb536be2f4)
[用 Retrofit 2 简化 HTTP 请求](https://academy.realm.io/cn/posts/droidcon-jake-wharton-simple-http-retrofit-2/)
[Android 网络框架之Retrofit2使用详解及从源码中解析原理 - 管满满 - 博客园](https://www.cnblogs.com/guanmanman/p/6085200.html)
[Making Retrofit Work For You by Jake Wharton - YouTube](https://www.youtube.com/watch?v=t34AQlblSeE)
[拆轮子系列：拆 Retrofit - Piasy的博客 | Piasy Blog](https://blog.piasy.com/2016/06/25/Understand-Retrofit/)
在接口方法和接口方法参数上的注解，通过这些注解表明该如何处理该网络请求；也就是说，注解都是注解在接口上的。
### 1.请求方法
* 每个方法都必须包括一个Http请求方法以及一个相对路径，这个Http请求方法以及相对路径是由注解来提供的。
* 请求的方法包括GET，POST，PUT，DELETE以及HEAD
* 相对路径则对应于请求方法注解的属性值，如@GET("users/list")
### 2.URL操作
* 通过替换块{id}以及方法参数可以动态的改变一个URL，这个方法参数需要使用@Path来注解；如
@GET("group_{id}_users")
Call<List<User>> groupList(@Path("id") int groupId);
* 当然也可以给URL添加查询参数
@GET("group_{id}_users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
* 如果有多个查询参数，那么也可使用
@GET("group_{id}_users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
### 3.Http请求体
可以将一个对象指定为http的请求体，如
@POST("users/new")
Call<User> createUser(@Body User user);
### 4.表单编码
* 如果是表单数据，那么这个时候每个参数需要使用@Field来注解 
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
* 如果是支持分组传输，那么使用@Part注解
 @Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
### 5.Http请求头操作
* 通过@Headers来设置静态的Http请求头
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();
* 当然你可以通过@Header来动态设置http请求头
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
对于设置多个消息头，你可以通过@HeaderMap来设置
@GET("user")
Call<User> getUser(@HeaderMap Map<String, String> headers)
### 6.同步和异步
可以同步或异步来执行call实例。每个实例只能被调用一次
## Retrofit的源码分析
1. 创建retrofit实例，在创建retrofit实例时需要设置baseUrl，convertFactory，callAdapter等，是通过构造者模式来创建的
2. 通过githubService=retrofit.create(GithubService.class)来创建GithubService实例
3. 调用githubService来获取一个请求，如Call类型或者Observable类型
4. 通过同步模式来调用这个call实例，call.execute()或者异步调用call.enqueue(Callback)

### InvocationHandler
调用接口实例的方法的时候，由于使用的是动态代理，所以所有的接口请求会被转移到InvocationHandler来。
在invocationHandler做了如下操作
* 获取ServiceMethod，对于同一个接口的同一个方法，只会创建一次，并会将这个实例缓存
* 通过serviceMethod和参数生成OkHttpCall对象
* 然后调用serviceMethod.adapt(okHttpCall)方法并返回接口调用结果，在这里调用结果是Call或者Observable。
#### 1. ServiceMethod
ServiceMethod中有
1. callFactory：主要用来生成okHttp的call，默认实现是OkHttpClient
2. callAdapter：根据需求将一个网络请求适配成Retrofit的call或者RxJava的Observable
3. responseConverter：将结果适配成Retrofit的Response
4. ParameterHandler：负责解析API定义时每个方法的参数，并在构造Http请求时设置请求参数
#### 2.生成OkHttpCall实例
#### 3.将OkHttpCall适配成Retrofit的call或者observable

![](Retrofit%E7%9A%84%E4%BD%BF%E7%94%A8/DC675112-AA45-45C4-A625-2D8E9557E303.png)
### OkHttpCall.execute()
1. 一个Retrofit的call只能执行一次
2. 调用createRawCall()生成一个OkHttp的call，其实质是通过serviceMethod.toCall(args)来生成的
3. 直接调用OkHttp的execute方法，并且将OkHttp的Respose转换成Retrofit的response，这个时候会调用serviceMethod的toResponse方法，这个方法会使用responseConverter



