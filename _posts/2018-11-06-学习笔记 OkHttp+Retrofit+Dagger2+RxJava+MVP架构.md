---
layout: post  
title:  "学习笔记| OkHttp+Retrofit+Dagger2+RxJava+MVP架构"  
date: 2018-11-06  
description: "一步一步地讲解各个框架特性及使用"
tag: Android开发
---

> 一口吃不成一个大胖子，一步一步地讲解各个框架特性及使用，再连接起来。

@[toc]
# OkHttp
```java
implementation 'com.squareup.okhttp3:okhttp:3.11.0'
```
## Header的设置

 - 使用header(name,value)来设置HTTP头的唯一值，如果请求中已经存在响应的信息那么直接替换掉。
- 使用addHeader(name,value)来补充新值，如果请求头中已经存在name的name-value，那么还会继续添加，请求头中便会存在多个name相同而value不同的“键值对”。
- 使用header(name)读取唯一值或多个值的最后一个值
- 使用headers(name)获取所有值

## GET & POST请求
```java
private String post() throws IOException {
		MediaType mediaType = MediaType.parse("text/x-markdown; charset=utf-8");
		String requestBody = "I am Renly.";
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url(NetConfig.REQUEST_URL)
                .header("Token","TokenValue")
                .addHeader("HeaderName","HeaderValue")
                .post(RequestBody.create(mediaType, requestBody)) // 默认是GET请求，GET请求可以不写
                .build();
//		   同步请求
        Response response = client.newCall(request).execute();
//         异步请求，此方法不适合该方法
//        Response response = client.newCall(request).enqueue(new Callback() {
//            @Override
//            public void onFailure(Call call, IOException e) {
//                Log.e(TAG,"getResponse fail " + e.getMessage());
//            }
//
//            @Override
//            public void onResponse(Call call, Response response) throws IOException {
//                Log.e(TAG,"getResponse success " + response.body().string());
//            }
//        });
        return response.body().string();
    }
```
## 拦截器-interceptor
OkHttp的拦截器链可谓是其整个框架的精髓，用户可传入的 **interceptor** 分为两类：
1. 一类是全局的 **interceptor**，该类 **interceptor** 在整个拦截器链中最早被调用，通过 **OkHttpClient.Builder#addInterceptor(Interceptor)** 传入；
2. 另外一类是非网页请求的 **interceptor** ，这类拦截器只会在非网页请求中被调用，并且是在组装完请求之后，真正发起网络请求前被调用，所有的 **interceptor** 被保存在 **List<Interceptor\> interceptors** 集合中，按照添加顺序来逐个调用，具体可参考 **RealCall#getResponseWithInterceptorChain()** 方法。通过 **OkHttpClient.Builder#addNetworkInterceptor(Interceptor)** 传入；

这里举一个简单的例子，例如有这样一个需求，我要监控App通过 OkHttp 发出的所有原始请求，以及整个请求所耗费的时间，针对这样的需求就可以使用第一类全局的 interceptor 在拦截器链头去做。
```java
OkHttpClient okHttpClient = new OkHttpClient.Builder()
        .addInterceptor(new LoggingInterceptor())
        .build();
Request request = new Request.Builder()
        .url("http://www.publicobject.com/helloworld.txt")
        .header("User-Agent", "OkHttp Example")
        .build();
okHttpClient.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        Log.d(TAG, "onFailure: " + e.getMessage());
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {
        ResponseBody body = response.body();
        if (body != null) {
            Log.d(TAG, "onResponse: " + response.body().string());
            body.close();
        }
    }
});
```
```java
public class LoggingInterceptor implements Interceptor {
    private static final String TAG = "LoggingInterceptor";
    
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();

        long startTime = System.nanoTime();
        Log.d(TAG, String.format("Sending request %s on %s%n%s",
                request.url(), chain.connection(), request.headers()));

        Response response =  chain.proceed(request);

        long endTime = System.nanoTime();
        Log.d(TAG, String.format("Received response for %s in %.1fms%n%s",
                response.request().url(), (endTime - startTime) / 1e6d, response.headers()));

        return response;
    }
}
```
![完整拦截器链](https://upload-images.jianshu.io/upload_images/3631399-164b722ab35ae9bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/432/format/webp)

-------------
# Retrofit
```java
implementation 'com.squareup.retrofit2:retrofit:2.4.0'
```
## Retrofit注解
- 请求方法
![请求方法](https://upload-images.jianshu.io/upload_images/1724103-db95c51539b62c96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/527/format/webp)

- 请求参数
![请求参数](https://upload-images.jianshu.io/upload_images/1724103-073abf80aacf492e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/741/format/webp)
- 请求标记
![请求标记](https://upload-images.jianshu.io/upload_images/1724103-4d09b5595bfb3291.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/671/format/webp)
## 请求姿势
### 1. 创建 接收服务器返回数据 的类
```java
public class Comment {
    private long id;
    private String body;
    private String user_id;
    ......
    }
```
### 2. 创建 用于描述网络请求 的接口
Retrofit将 **Http请求** 抽象成 **Java接口**：采用 **注解** 描述网络请求参数和配置网络请求参数 

> 用 动态代理 动态 将该接口的注解“翻译”成一个 Http 请求，最后再执行 Http 请求 
注：接口中的每个方法的参数都需要使用注解标注，否则会报错
```java
public interface APi {
	// @GET注解的作用:采用Get方法发送网络请求
	// "comments"用于在创建Retrofit实例时拼接网络请求的URL
    @GET("comments")
    Call<List<Comment>> getComments(@Header("Authorization") String authorization, @Query("page")int page);
    // getComments() = 接收网络请求数据的方法
    // 其中返回类型为Call<*>，*是接收数据的类（即上面定义的Comment类）

	@Headers("Authorization: authorization")
    @POST("login")
    Call<ResponseBody> doLogin(@Query("email") String email, @Query("password") String password);
    // 如果想直接获得Responsebody中的内容，可以定义网络请求返回值为Call<ResponseBody>

	// Header:
	// 以上的效果是一致的。
    // 区别在于使用场景和使用方式
	// 1. 使用场景：@Header用于添加不固定的请求头，@Headers用于添加固定的请求头
	// 2. 使用方式：@Header作用于方法的参数；@Headers作用于方法
}
```
### 3. 创建 Retrofit的实例 并 发起请求
只演示post请求,get请求同理
```java
private void TestRetrofit() {
        OkHttpClient client = new OkHttpClient.Builder().build();
        // 创建Retrofit实例
        Retrofit retrofit = new Retrofit.Builder()
		        .baseUrl(NetConfig.BASE_URL) //网络请求完整的URL = 通过实例化时.baseUrl()设置+网络请求接口的注解设置
				.addConverterFactory(GsonConverterFactory.create()) // 设置数据解析器
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) // 支持RxJava平台
                .client(client)
                .build();
        APi mApi = retrofit.create(APi.class);
        
        //发送网络请求(异步)
        retrofit2.Call<ResponseBody> call = mApi.doLogin("testUser@qq.com", "testPwd");
        call.enqueue(new retrofit2.Callback<ResponseBody>() {
            @Override
            public void onResponse(retrofit2.Call<ResponseBody> call, retrofit2.Response<ResponseBody> response) {
            	//请求成功时回调
                Log.e("print","onResponse");
                try {
                    tv.setText(response.body().string());
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void onFailure(retrofit2.Call<ResponseBody> call, Throwable t) {
            	//请求失败时候的回调
                Log.e("print","onFailure " + t.getMessage());
            }
        });
        // 发送网络请求（同步）
        Response<ResponseBody> response = call.execute();
    }
```
### 4. 关于数据解析器（Converter）
```java
//设置数据解析器
.addConverterFactory(GsonConverterFactory.create())
```
这个有啥用？这句话的作用就是使得来自接口的json结果会自动解析成定义好了的字段和类型都相符的json对象接受类。在Retrofit 2.0中，Package 中已经没有Converter了,所以，你需要自己创建一个Converter， 不然的话Retrofit只能接收字符串结果，你也只能拿到一串字符，剩下的json转换的活还得你自己来干。所以，如果你想接收json结果并自动转换成解析好的接收类，必须自己创建Converter对象，然后使用addConverterFactory把它添加进来！
Retrofit支持多种数据解析方式，在使用时注意需要在Gradle添加依赖：
| 数据解析器 | Gradle依赖 |
|--|--|
| Gson	| com.squareup.retrofit2:converter-gson:2.0.2 |
| Jackson | com.squareup.retrofit2:converter-jackson:2.0.2 |
| Simple XML | com.squareup.retrofit2:converter-simplexml:2.0.2 |
| Protobuf | com.squareup.retrofit2:converter-protobuf:2.0.2 | 
| Moshi | com.squareup.retrofit2:converter-moshi:2.0.2 | 
| Wire | com.squareup.retrofit2:converter-wire:2.0.2 | 
| Scalars | com.squareup.retrofit2:converter-scalars:2.0.2 |
### 5. 关于网络请求适配器（CallAdapter）
```java
//设置网络请求适配器
.addCallAdapterFactory(RxJava2CallAdapterFactory.create())
```
| 网络请求适配器 | Gradle依赖 |
|--|--|
| Guava | com.squareup.retrofit2:adapter-guava:2.0.2 |
| Java8 | com.squareup.retrofit2:adapter-java8:2.0.2 |
| RXJava | com.squareup.retrofit2:adapter-rxjava:2.0.2 |
## 更多
注：addConverterFactory是有先后顺序的，如果有多个ConverterFactory都支持（就是大神所谓的我可以处理）同一种类型，那么就是只有第一个才会被使用，而GsonConverterFactory是不判断是否支持的（直接返回Converter）

**附时序图**
![时序图](https://upload-images.jianshu.io/upload_images/6163786-03bc66bb37973344.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**推荐阅读：**
[这是一份很详细的 Retrofit 2.0 使用教程（含实例讲解）](https://link.jianshu.com/?t=http%3A%2F%2Fblog.csdn.net%2Fcarson_ho%2Farticle%2Fdetails%2F73732076)
[Retrofit分析-漂亮的解耦套路](https://www.jianshu.com/p/45cb536be2f4)

Retrofit Javadoc:[Retrofit官方 API](https://square.github.io/retrofit/2.x/retrofit/)

--------------
# RxJava
```java
implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
implementation 'io.reactivex.rxjava2:rxjava:2.2.3'
```
由于这部分知识过于庞大，自知实力没法提炼出精髓就不班门弄斧了，这里推荐几篇文章，这些文章对我理解RxJava有很大的帮助，推荐结合看才能深入了解。
[优美的异步 --- RxAndroid](https://www.jianshu.com/p/7eb5ccf5ab1e)
[给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083)
[RxJava源码分析与实战](https://blog.csdn.net/johnny901114/article/category/6249462)  关于RxJava的专栏，适于深入了解（像是flatMap,retryWhen操作符）

接下来会开一个项目来写OkHttp+Retrofit+RxJava

--------------
# Dagger2
```java
implementation 'com.google.dagger:dagger:2.19'
annotationProcessor 'com.google.dagger:dagger-compiler:2.19'
```
## 关键的注解
**@Inject**
这个注解是用来说明该注解下方的属性或方法需要依赖注入。（如果使用在类构造方法上，则该类也会被注册在DI容器中作为注入对象。很重要，理解这个，就能理解Presenter注入到Activity的步骤！）

**@Provider**
在@Module注解的类中，使用@Provider注解，说明提供依赖注入的具体对象

**@Component**
简单说就是，可以通过Component访问到Module中提供的依赖注入对象。假设，如果有两个Module，AModule、BModule，如果Component只注册了AModule，而没有注册BModule，那么BModule中提供的对象，无法进行依赖注入！

**@SubComponent**
该注解从名字上就能知道，它是子Component，会完全继承父Component的所有依赖注入对象！

**@Sigleton**
被注解的对象，在App中是单例存在的！

**@Scope**
用来标注依赖注入对象的适用范围。

**@Named(@)**
因为Dagger2 的以来注入是使用类型推断的，所以同一类型的对象就无法区分，可以使用@Named注解区分同一类型对象，可以理解为对象的别名！

--------------
# MVP架构
## MVP是什么
我们所说的**mvp架构**，是google开源的一个**设计模式**，目的是为了将代码更加优雅清晰的呈现出来，废话也不多说，直接分析

**M**：M层，也就是我们在程序中经常出现的**model层**，他的功能就是**处理数据**，其他任务不由他来接手。
**V**：V层，我们的**view层**，也就是**显示数据**的地方，我们在得到数据之后，把数据传递给view层，通过他来显示数据。同时，view层的点击事件等处理会在这里出现，但真正的数据处理不是在这里，而是在model层中处理。
**P**：P层，也就是**Presenter层**，他是我们mvp架构中的中间人，通过p层的连接，让我们可以是M层和V层进行通信。**M层在获取到数据之后，把它交给P层，P层在交给View层**，同样，View层的点击事件等处理通过P层去通知M层，让他去进行数据处理。
