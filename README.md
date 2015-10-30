# 从 Volley 源代码开始，一步一步学习网络请求

> 从 2015.10.30 开始研读 Volley 代码，边学习、边记录、边分享。

## 前言
(待补充)

我们开始使用 Volley，写的第一行代码就是

`RequestQueue requestQueue = Volley.newRequestQueue(this);`

那么，我们就从 Volley 的构造函数开始，一步一步来看看究竟是怎么回事。学习过程中，请对照源码来看，效率会更高。

### Volley.java

Volley 这个类只有两个重载的静态方法：

```
public static RequestQueue newRequestQueue(Context context)
public static RequestQueue newRequestQueue(Context context, HttpStack stack)
```

第一个方法很简单，直接调用了第二个方法，只不过是第二个参数传递一个默认的 `null`

在第二个方法中，第一个参数不做过多解释，我们来看看第二个参数 [HttpStack](https://github.com/VegeChou/Volley#jump)

下面我们继续分析第二个方法：

```
1. public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
2.     File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);
3.
4.     String userAgent = "volley/0";
5.     try {
6.         String packageName = context.getPackageName();
7.         PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
8.         userAgent = packageName + "/" + info.versionCode;
9.     } catch (NameNotFoundException e) {
10.    }
11.
12.    if (stack == null) {
13.        if (Build.VERSION.SDK_INT >= 9) {
14.            stack = new HurlStack();
15.        } else {
16.            // Prior to Gingerbread, HttpUrlConnection was unreliable.
17.            // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
18.             stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
19.        }
20.     }
21.
22.     Network network = new BasicNetwork(stack);
23.
24.     RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
25.     queue.start();
26.
27.     return queue;
28. }
```

第 2 行：定义了 RequestQueue 缓存的目录，关于 RequestQueue 的缓存后面再说

第 4-10 行：定义了 UserAgent，如果获取包名失败，为默认值`volley/0`；如果获取成功，则为 `包名/版本号`。例如 `com.android.xxx/12`

> 这里涉及到一个网络通信的知识 `UserAgent`，关于 UserAgent 的介绍和作用请看[这里](http://www.cnblogs.com/tonytonglx/articles/2063110.html)

第 12-20 行：如果传入的 HttpStack 为空，Volley 会根据系统版本不同，创建对应的 HttpStack 对象。

> 1. 如果 API >= 9，采用的是基于 HttpURLConnection 的 HurlStack，如果 API < 9，采用的是基于 HttpClient 的 HttpClientStack。具体关于这两种方式的为什么这么选择，请参考[这里](http://www.trinea.cn/android/android-http-api-compare/)

> 2. HttpStack 如果不为空，那么就需要用户自己实现 HttpStack 这个接口，相当于可以完全自定义网络请求模块。Volley 的模块化、高自由度由此开始体现。

第 22 行：使用 HttpStack 构建基础网络通信模块

第 24、25行：使用缓存模块和网络通信模块构造 `RequestQueue`，启动 `RequestQueue`

> 我们已经知道，Volley 对于处理图片缓存，有较高的性能。目前虽然不知道 `DiskBasedCache` 如何使用，但一定与此相关，后面再看。

至此，Volley.java 解析已经结束，下面我们详细看看 `HurlStack` 和 `HttpClientStack` 是如何实现的。

### <span id="jump">HttpStack.java</span>

`HurlStack` 和 `HttpClientStack` 都实现了 HttpStack 的接口







