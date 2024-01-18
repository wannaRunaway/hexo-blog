---
title: Android-okhttp3单独使用
tags: Android
---
 最近使用了okhttp3 + mvvm的架构框架，也了解了volley，httpclient，async-http等框架，这些框架之前有的使用过，比起okhttp3来说，okhttp3有各种自定义拦截器，这里介绍下okhttp的各种方法吧。



## get请求

> 构造request时候，直接吧参数拼成name=xx&pwd=xx这种表单格式放在?后面。

```
request = new Request.Builder().url(API.baseurl + url + "?" + stringJson).header("ACCESS_TOKEN", Utils.getToken(activity)).build();
```



## post请求

> requestbody.create(stringjson,JSON)，stringjson是json格式的请求参数，后面的JSON是互联网媒体类型。

```
if (stringJson == null) {
	body = RequestBody.create("", JSON);
} else {
	body = RequestBody.create(stringJson, JSON);
}
request = new Request.Builder().url(API.baseurl + url).post(body).header("ACCESS_TOKEN", Utils.getToken(activity)).build();
```



## 上传媒体类型

> 也叫做MIME类型，在Http协议消息头中，使用Content-Type来表示。具体请求中的媒体类型信息。用于定义网络文件的类型和网页的编码，决定文件接收方将以什么形式、什么编码读取这个文件。

 text/html：HTML格式

 text/pain：纯文本格式

 image/jpeg：jpg图片格式

 application/json：JSON数据格式

 application/octet-stream：二进制流数据（如常见的文件下载）

 application/x-www-form-urlencoded：form表单encType属性的默认格式，表单数据将以key/value的形式发送到服务端

 multipart/form-data：表单上传文件的格式



## 请求返回

> 异步返回

```
enqueue client.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
            }
            @Override
            public void onResponse(Call call, Response response) throws IOException {
            }
        });
```



## 自定义拦截器

> 初始化okhttpclient并设置默认信任所有证书，添加拦截器，为了不断更新token加在请求头给服务端，所以我的初始化代码是这样写的.

```
private OkHttpClient okhttpclient(Activity activity) {
        try {
            if (client == null) {
                client = new OkHttpClient.Builder().
                        sslSocketFactory(createSSLSocketFactory(), new TrustAllCerts())
                        .hostnameVerifier(new TrustAllHostnameVerifier()).addInterceptor(interceptor(activity))
                        .build();
            }
            return client;
        } catch (Exception e) {
            Utils.log("", "", "okhttp https连接问题");
            return null;
        }
    }
```

> 拦截器就是为了拿到服务端response里面的token和打印各种请求和返回，代码如下：

```
public class HttpLogInterceptor implements Interceptor {
    //    private static final String TAG = HttpLogInterceptor.class.getSimpleName();
    private final Charset UTF8 = Charset.forName("UTF-8");
    private Activity activity;
    public HttpLogInterceptor(Activity activity) {
        this.activity = activity;
    }
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        RequestBody requestBody = request.body();
        String body = null;
        if (requestBody != null) {
            Buffer buffer = new Buffer();
            requestBody.writeTo(buffer);
            Charset charset = UTF8;
            MediaType contentType = requestBody.contentType();
            if (contentType != null) {
                charset = contentType.charset(UTF8);
            }
            body = buffer.readString(charset);
        }
        Log.d("xuedi", "发送请求: method：" + request.method()
                + "\nurl：" + request.url()
                + "\n请求头：" + request.headers()
                + "\n请求参数: " + body);
        long startNs = System.nanoTime();
        Response response = chain.proceed(request);
        long tookMs = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startNs);
        if (response.header("ACCESS_TOKEN")!=null) {
            UserLogin.getInstance().setToken(response.header("ACCESS_TOKEN"));
            SharePref.put(activity, API.token, response.header("ACCESS_TOKEN"));
        }
        ResponseBody responseBody = response.body();
        String rBody;
        BufferedSource source = responseBody.source();
        source.request(Long.MAX_VALUE);
        Buffer buffer = source.buffer();
        Charset charset = UTF8;
        MediaType contentType = responseBody.contentType();
        if (contentType != null) {
            try {
                charset = contentType.charset(UTF8);
            } catch (UnsupportedCharsetException e) {
                e.printStackTrace();
            }
        }
        rBody = buffer.clone().readString(charset);
        Log.d("xuedi", "收到: method：" + request.method()+
                "\n响应header："+response.header("ACCESS_TOKEN")
                +"\n响应url:"+response.request().url()
                + "\n响应body: " + rBody);
        return response;
    }
```

> 这里面就是重写的intercept(Chain chain),chain.request和chain.response拿到就可以打印了，这是Chain接口方法：

```
public interface Chain {
public abstract fun call(): okhttp3.Call
public abstract fun connectTimeoutMillis(): kotlin.Int
public abstract fun connection(): okhttp3.Connection?
public abstract fun proceed(request: okhttp3.Request): okhttp3.Response
public abstract fun readTimeoutMillis(): kotlin.Int
public abstract fun request(): okhttp3.Request
public abstract fun withConnectTimeout(timeout: kotlin.Int, unit: java.util.concurrent.TimeUnit): okhttp3.Interceptor.Chain
public abstract fun withReadTimeout(timeout: kotlin.Int, unit: java.util.concurrent.TimeUnit): okhttp3.Interceptor.Chain
public abstract fun withWriteTimeout(timeout: kotlin.Int, unit: java.util.concurrent.TimeUnit): okhttp3.Interceptor.Chain
public abstract fun writeTimeoutMillis(): kotlin.Int
}
```

拿到保存在本地，然后返回给服务端，这就是一次完整的请求。当然还有多类型上传，formbody上传等等。官方文档：

https://square.github.io/okhttp