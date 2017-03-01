https://github.com/google/volley

###Volley学习笔记

 虽然更倾向于retrofit + okhttp，项目中使用Volley近一年了，从学习的角度扒一扒Volley代码吧
 
 关于Volley的整体结构和流程不再赘述，直接上代码
 
#####Volley.java
`Volley.java`用来创建 `RequestQueue`，有两个用来重载静态方法和一个常量（默认的磁盘缓存目录），核心代码如下：

      public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
          File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);
          String userAgent = "volley/0";
          try {
              String packageName = context.getPackageName();
              PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
              userAgent = packageName + "/" + info.versionCode;
          } catch (NameNotFoundException ignored) {
          }
          if (stack == null) {
              if (Build.VERSION.SDK_INT >= 9) {
                  stack = new HurlStack();
              } else {
                  stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
              }
          }
          Network network = new BasicNetwork(stack);
          RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
          queue.start();
          return queue;
      }
      
- 1.创建缓存目录
- 2.生成内容为`包名/版本号`的UA，默认为"volley/0"
- 3.如果未指定stack，在api9以上的设备上使用基于HttpURLConnection的stack，更旧是设备上则是用HttpClient实现的stack。传入的HttpStack为接口类型，因此可以使用自己的HttpStack实现（如okhttp）。
- 4.创建BasicNetwork对象和DiskBasedCache对象
- 5.通过4中的对象构造一个RequestQueue，并调用start()方法后将其返回