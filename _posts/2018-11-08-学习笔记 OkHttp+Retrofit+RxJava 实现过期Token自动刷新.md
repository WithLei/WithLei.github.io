---
layout: post  
title:  "学习笔记| OkHttp+Retrofit+RxJava 实现过期Token自动刷新"  
date: 2018-11-08  
description: "各自职责：Retrofit 负责 请求的数据 和 请求的结果，使用 接口的方式 呈现，OkHttp 负责请求的过程，RxJava 负责异步，各种线程之间的切换"
tag: Android开发
---

> 在经历了OkHttp、Retrofit、RxJava的学习后，终于可以开始写代码rua！

由于网络上安利这几款火的不行的框架的博客实在是**太多太多太多**了，介绍、优缺点之类的废话就不多说了，这里只介绍下关系。
- Retrofit：Retrofit是Square公司开发的一款针对Android 网络请求的框架（底层默认是基于OkHttp 实现）。
- OkHttp：也是Square公司的一款开源的网络请求库。
- RxJava ："a library for composing asynchronous and event-based programs using observable sequences for the Java VM"（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）。RxJava使异步操作变得非常简单。

各自职责：**Retrofit 负责 请求的数据 和 请求的结果，使用 接口的方式 呈现，OkHttp 负责请求的过程，RxJava 负责异步，各种线程之间的切换**。

# 一、添加依赖
在build.gradle文件中添加如下配置：
```java
	// rxjava
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
    implementation 'io.reactivex.rxjava2:rxjava:2.2.3'
	// retrofit
    implementation 'com.squareup.retrofit2:retrofit:2.4.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.3.0'
    implementation 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
    // okhttp
    implementation 'com.squareup.okhttp3:okhttp:3.11.0'
    //gson
    implementation 'com.google.code.gson:gson:2.8.5'
```
# 二、更新token模块
有关JWT的知识可以看一下大神的博客：[JSON Web Token 入门教程 - 阮一峰](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
**刷新tokenAPI**
![刷新tokenAPI](https://img-blog.csdnimg.cn/20181117114158197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODk1Mzc5,size_16,color_FFFFFF,t_70)
## 实现思路
利用 Observale 的 retryWhen 的方法，识别 token 过期失效的错误信息，此时发出刷新 token 请求的代码块，完成之后更新 token，这时之前的请求会重新执行，但将它的 token 更新为最新的。另外通过代理类对所有的请求都进行处理，完成之后，我们只需关注单个 API 的实现，而不用每个都考虑 token 过期，大大地实现解耦操作。
## Token 存储模块
存储token使用的是SharedPreferences + 单例模式 避免并发请求行为
```java
public class Store {
    private SharedPreferences mStore;
	// 单例模式
    private Store(){
        mStore = App.getContext().getSharedPreferences(App.MY_SP_NAME, Context.MODE_PRIVATE);
    }

    public static Store getInstance() {
        return Holder.INSTANCE;
    }

    private static final class Holder {
        private static final Store INSTANCE = new Store();
    }

    public void setToken(String token) {
        mStore.edit().putString(App.USER_TOKEN_KEY, token).apply();
    }

    public String getToken() {
        return mStore.getString(App.USER_TOKEN_KEY, "");
    }
}
```
## 完整的Token请求过程
我将token请求提取到retrofit service层方便全局调用，使用直接返回一个observable对象，对其订阅在观察者里实现携带token的请求数据操作。*并且这里特意没有用lambda表达式写，对于理解会方便很多*
```java
	/**
     * 获取新的Token
     */
    private static final String ERROR_TOKEN = "error_token";
    private static final String ERROR_RETRY = "error_retry";
    public static Observable<String> getNewToken() {
        return Observable.defer(new Callable<ObservableSource<String>>() {
            @Override
            public ObservableSource<String> call() throws Exception {
                OkHttpClient client = new OkHttpClient();
                MediaType mediaType = MediaType.parse("text/x-markdown; charset=utf-8");
                String requestBody = "";
                Request request = new Request.Builder()
                        .url(NetConfig.BASE_GETNEWTOKEN_PLUS)
                        .header("Authorization", "Bearer " + Store.getInstance().getToken())
                        .post(RequestBody.create(mediaType, requestBody))
                        .build();
                Log.e("print","发起Token请求");
                return Observable.just(client.newCall(request).execute().body().string());
            }
        })
                // Token判断
                .flatMap(new Function<String, ObservableSource<String>>() {
                    @Override
                    public ObservableSource<String> apply(String s) throws Exception {
                        return Observable.create(new ObservableOnSubscribe<String>() {
                            @Override
                            public void subscribe(ObservableEmitter<String> emitter) {
                                JSONObject obj = JSON.parseObject(s);
                                if (obj.getInteger("code") != 20000) {
                                    emitter.onError(new Throwable(ERROR_RETRY));
                                } else {
                                    String token = obj.getString("result");
                                    Store.getInstance().setToken(token);
                                    emitter.onNext(token);
                                }
                            }
                        });
                    }
                })
                // flatMap若onError进入retrywhen，否则onNext()
                .retryWhen(new Function<Observable<Throwable>, ObservableSource<?>>() {
                    private int mRetryCount = 0;

                    @Override
                    public ObservableSource<?> apply(Observable<Throwable> throwableObservable) {
                        return throwableObservable.flatMap(new Function<Throwable, ObservableSource<?>>() {
                            @Override
                            public ObservableSource<?> apply(Throwable throwable) throws Exception {
                                if (mRetryCount++ < 3 && throwable.getMessage().equals(ERROR_TOKEN))
                                    return Observable.error(new Throwable(ERROR_RETRY));
                                return Observable.error(throwable);
                            }
                        });
                    }
                });
    }
```
为了方便大家阅读，我把所有的逻辑都写在了一整个调用链里，整个调用链分为三个部分：

 1. **defer**：读取缓存中的token信息，这里调用了TokenLoader中读取缓存的接口，而这里使用defer操作符，是为了在重订阅时，重新创建一个新的Observable，以读取最新的缓存token信息，其原理图如下：
 ![defer原理图](https://upload-images.jianshu.io/upload_images/1949836-75602a96deed77e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
 2. **flatMap**：通过token信息，请求必要的接口。
 3. **retryWhen**：使用重订阅的方式来处理token失效时的逻辑，这里分为三种情况：重试次数到达，那么放弃重订阅，直接返回错误；请求token接口，根据token请求的结果决定是否重订阅；其它情况直接放弃重订阅。

# 三、依赖于Token的数据获取模块
在这里，我选择抽离出项目中的**获取用户信息模块**进行代码重构演示
**获取用户信息API**
![获取个人信息API_1](https://img-blog.csdnimg.cn/20181117112155634.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODk1Mzc5,size_16,color_FFFFFF,t_70)
![获取个人信息API_2](https://img-blog.csdnimg.cn/20181117112218599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODk1Mzc5,size_16,color_FFFFFF,t_70)
我选择在将个人信息请求写在了观察者的方法里，再次嵌套一个链式结构
```java
public synchronized void getUserAvator() {
		// 创建被观察者的实例
        Observable<String> observable = RetrofitService.getNewToken();
		// 定义观察者
        DisposableObserver<String> observer = new DisposableObserver<String>() {
            @Override
            public void onNext(String s) {
            	// 发起用户信息请求
                Observable.create((ObservableOnSubscribe<String>) emitter -> {
                    OkHttpClient client = new OkHttpClient();
                    Request request = new Request.Builder()
                            .url(NetConfig.BASE_USERDETAIL_PLUS)
                            .header("Authorization", "Bearer " + Store.getInstance().getToken())
                            .get()
                            .build();
                    String response = client.newCall(request).execute().body().string();
                    emitter.onNext(response);
                })
                        .subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread())
                        .subscribe(responseString -> {
                            if (!responseString.contains("result")){
                                printLog("HomeFragment_getAvatar_subscribe:获取用户信息出错 需要处理");
                                return;
                            }
                            JSONObject jsonObject = JSON.parseObject(responseString);
                            String path = "";
                            JSONObject obj = JSON.parseObject(jsonObject.getString("result"));
                            path = obj.getString("avatar");
                            Picasso.get()
                                    .load(path)
                                    .placeholder(R.drawable.image_placeholder)
                                    .into(ciHomeImg);
                        }, throwable -> printLog("HomeFragment_getAvatar_subscribe_onError:" + throwable.getMessage()));

            }

            @Override
            public void onError(Throwable e) {
                Log.e("print", "HomeFragment_getUserAvator_onError: " + e.getMessage());
            }

            @Override
            public void onComplete() {

            }
        };
        // 进行订阅
        observable.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(observer);
    }
```
# 四、总结
这种实现方法实际上每次从服务器获取信息执行顺序是：更新Token → 获取数据信息，这样会向服务器产生过多不必要的请求，加大服务器的负担。所以正确的请求姿势应该是：发起数据请求 → 返回token过期信息 → 发送更新token请求 → 返回new token → 再次发起数据请求，可以有效地减轻服务器的负担，可以在上面的基础上再次封装token service服务达到这样的效果。
# 五、更多
在 coding 前参考了很多博客，推荐几篇好的文章
[defer操作符实现代码支持链式调用 - Chiclaim](https://blog.csdn.net/johnny901114/article/details/52597643)
[retryWhen操作符实现错误重试机制 - Chiclaim](https://blog.csdn.net/johnny901114/article/details/51539708)
[在 token 过期时，刷新过期 token 并重新发起请求 - 泽毛](https://www.jianshu.com/p/e88e61e1a721)