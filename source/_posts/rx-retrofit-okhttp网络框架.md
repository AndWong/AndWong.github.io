---
title: rx+retrofit+okhttp网络框架
date: 2017-03-06 22:30:35
tags: Android
---
首先导入相关的包
compile 'com.google.code.gson:gson:2.5'
compile 'com.squareup.okhttp3:okhttp:3.3.1'
compile 'io.reactivex:rxjava:1.1.9'
compile 'io.reactivex:rxandroid:1.2.1'
compile 'com.squareup.retrofit2:retrofit:2.1.0'
compile 'com.squareup.retrofit2:converter-gson:2.1.0'
compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
compile 'com.squareup.retrofit2:retrofit-converters:2.1.0'
compile 'com.squareup.okhttp3:logging-interceptor:3.4.1'

1.对应的数据模型,采用范型
```
public class BaseModel<T> implements Serializable {
    public int status;
    public String message;
    public T data;

    public BaseModel() {
    }

    public BaseModel(int status, String message, T data) {
        this.status = status;
        this.message = message;
        this.data = data;
    }
}
```
2.相关接口,和服务端的接口对应
```
public interface DataApi {
    @GET(URL)
    Observable<BaseModel<T>> fetchData(@QueryMap Map<String, Object> params);
}
```
3.网络配置,请求等基础类
```
public class BaseService {
    private static final String TAG = "BaseService";
    protected static final String BASE_URL = "baseurl";
    protected Retrofit retrofit;
    protected OkHttpClient client;
    private HttpLoggingInterceptor logging; //日志类

    public BaseService() {
        initHttpLoggingInterceptor();
    }
    /**
     * 构建Retrofit
     **/
    protected Retrofit getRetrofit() {
        if (retrofit == null) {
            retrofit = new Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create())
                    .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                    .client(getOkHttpClient())
                    .build();
        }
        return retrofit;
    }
    /**
     * 构建Okhttp
     **/
    protected OkHttpClient getOkHttpClient() {
        if (client == null) {
            client = getOkHttpBuilder().build();
        }
        return client;
    }

    private OkHttpClient.Builder getOkHttpBuilder() {
        OkHttpClient.Builder builder = new OkHttpClient.Builder()
                .connectTimeout(30, TimeUnit.SECONDS)
                .addInterceptor(logging);
        return builder;
    }

    /**
     * 初始化 HTTP日志打印拦截
     */
    private void initHttpLoggingInterceptor() {
        logging = new HttpLoggingInterceptor(new HttpLoggingInterceptor.Logger() {
            @Override
            public void log(String message) {
                LogUtil.d(TAG, "whd >> MSG : " + message);
            }
        });
        logging.setLevel(HttpLoggingInterceptor.Level.BODY);
    }

    /**
     * 支持使用HTTPS请求
     *
     * @return
     */
    public Retrofit getHTTPSRetrofit() {
        OkHttpClient.Builder builder = getOkHttpClient().newBuilder();
        setClientSSL(builder);
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl(BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .client(builder.build())
                .build();
        return retrofit;
    }

    /**
     * 支持HTTPS的 SSL
     *
     * @param builder
     */
    private void setClientSSL(OkHttpClient.Builder builder) {
        TrustManager[] trustManagers = new TrustManager[]{(TrustManager) new HTTPSTrustManager()};
        try {
            SSLContext sslContext = SSLContext.getInstance("SSL");
            sslContext.init(null, trustManagers, new java.security.SecureRandom());
            SSLSocketFactory sslSocketFactory = sslContext.getSocketFactory();
            builder.sslSocketFactory(sslSocketFactory);
        } catch (Exception e) {
            LogUtil.d(TAG, "setClientSSL() :" + e.getMessage());
        }
    }
}
```
```
/**
 * 使用它跳过HTTPS的SSL认证
 */
public class HTTPSTrustManager {
    private static TrustManager[] trustManagers;
    private static final X509Certificate[] _AcceptedIssuers = new X509Certificate[0];

    public HTTPSTrustManager() {
    }

    public void checkClientTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {
    }

    public void checkServerTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {
    }

    public boolean isClientTrusted(X509Certificate[] chain) {
        return true;
    }

    public boolean isServerTrusted(X509Certificate[] chain) {
        return true;
    }

    public X509Certificate[] getAcceptedIssuers() {
        return _AcceptedIssuers;
    }
}
```
对应接口的一个请求Service类,继承BaseService
```
public class DataService extends BaseService {

    public Observable<BaseModel<T>> fetchData() {
        Map<String, Object> params = new ArrayMap<>(1);
        params.put(key, value);
        return getRetrofit().create(DataApi.class).fetchData(params);
    }

   /**
    * 添加拦截器,
    * 可以获取请求的数据和返回的数据.
    * 不进行数据处理的话可以不用重写该方法
    **/
    @Override
    protected OkHttpClient getOkHttpClient() {
        OkHttpClient client = super.getOkHttpClient();
        return client.newBuilder().addInterceptor(new DataInterceptor()).build();
    }
}
```
4.自定义拦截器
```
public class DataInterceptor implements Interceptor {
    private static final String TAG = "DataInterceptor";
    private static final Charset UTF8 = Charset.forName("UTF-8");

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        Request.Builder builder = request.newBuilder();
        Request newRequest = builder
                .addHeader("Accept", "application/json")
                .addHeader("Content-Type", "application/x-www-form-urlencoded")
                .addHeader("x-sdk-version", VersionUtils.SDK_VERSION)
                .addHeader("x-app-version", VersionUtils.getAppVersionName(App.sContext))
                .build();
        Response response = chain.proceed(newRequest);
        return buildResponse(response);
    }

    /**
     * 重新构造Response
     *
     * @param response
     * @return
     * @throws IOException
     */
    private Response buildResponse(final Response response) throws IOException {
        String result = decryptionResponse(response);
        return response.newBuilder().body(ResponseBody.create(response.body().contentType(), result)).build();
    }

    /**
     * 解密BODY
     *
     * @param response
     * @return
     */
    private String decryptionResponse(Response response) throws IOException {
        String data = null;
        ResponseBody responseBody = response.body();
        long contentLength = responseBody.contentLength();
        BufferedSource source = responseBody.source();
        source.request(Long.MAX_VALUE); // Buffer the entire body.
        Buffer buffer = source.buffer();

        Charset charset = UTF8;
        MediaType contentType = responseBody.contentType();
        if (contentType != null) {
            charset = contentType.charset(UTF8);
        }
        if (contentLength != 0) {
            String encrypStr = buffer.clone().readString(charset);
            BaseModel baseModel = JsonUtil.getInstance().fromJson(encrypStr, BaseModel.class);
            LogUtil.d(TAG, "加密内容 : " + wiFiBaseModel.data.toString());
            baseModel.data = JsonUtil.getInstance().fromJson(AesUtils.decrypt(baseModel.data.toString()), Object.class);
            data = JsonUtil.getInstance().toJson(baseModel);
            LogUtil.d(TAG, "解密内容 : " + data);
        }
        return data;
    }
}
```
5.json解析
```
/**
 * Json工具类
 * Created by wong on 17-3-6.
 */
public class JsonUtil {
    private static final String TAG = "JsonUtil";
    private static Gson gson = null;
    private static JsonUtil jsonUtil = null;

    private JsonUtil() {
        gson = new GsonBuilder().serializeNulls().create();
    }

    public static JsonUtil getInstance() {
        if (jsonUtil == null) {
            jsonUtil = new JsonUtil();
        }
        return jsonUtil;
    }

    /**
     * 将对象转换为JSON字符串
     *
     * @param obj
     * @return
     */
    public String toJson(Object obj) {
        return gson.toJson(obj);
    }

    /**
     * 将JSON字符串转化为对象
     *
     * @param s
     * @param cls
     * @return
     */
    public <T> T fromJson(String s, Type cls) {
        return gson.fromJson(s.trim(), cls);
    }

}
```
6.获取Service来进行相关操作
```
public class HttpManager {
    public static volatile HttpManager instance;
    public DataService dataService;

    private HttpManager() {

    }

    public static HttpManager getInstance() {
        if (null == instance) {
            synchronized (HttpManager.class) {
                if (null == instance) {
                    instance = new HttpManager();
                }
            }
        }
        return instance;
    }

    public DataService getDataService() {
        if (null == dataService) {
            dataService = new DataService();
        }
        return dataService;
    }

}
```
7.使用示例,创建一个IntentService进行网络请求
```
public class DataRequestService extends IntentService {

    public DataRequestService() {
        super("DataRequestService");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        HttpManager.getInstance().getDataService().fetchData()
                .subscribeOn(Schedulers.io())
                .subscribe(new Subscriber<BaseModel<T>>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(BaseModel<T> baseModel) {

                    }
                });
    }
}
```
