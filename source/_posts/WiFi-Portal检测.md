---
title: WiFi Portal检测
date: 2016-04-11 21:35:53
tags: Android
---
WiFi连接上一个开放热点，如何判断是否需要登录认证？

首先当我们发送一个request请求时，我们得到的response总是会携带一个HTTP状态码（除非请求超时）。WiFi portal同样也要遵循这样的规则，WiFi portal拦截网络请求时，返回给我们一个response（内容是登陆页面）。我们的这次请求虽然被拦截了，但是无疑他是一个成功的请求，也就是说response携带的状态码应该是 200 。这时，我们与服务器端商定一个请求链接http ://www.xx.com/genera_204，固定返回一个状态码 204。当网络状态发生变化时候，我们就去请求这个链接。如果我们的response携带的状态码是204说明我们请求成功，如果我们得到的是200，说明需要进行WiFi 验证。
这个方法来自Android4.0.1AOSP源码 WifiWatchdogStateMachine#isWalledGardenConnection()。

对于个人开发者或者“小厂商”不建议在我们自己的服务器上处理http ://www.xxx.com/generate_204，除非你家的服务器很稳定，基本上不出现问题。这时候我们可以考虑下“大厂商”是否已经有类似的功能。像UC啊什么的。他们都能自动提醒你网络需要登陆，他们也是使用了同样的方式，至于链接地址，就靠大家自己了。

代码如下：
```
import java.io.IOException;
import java.net.HttpURLConnection;
import java.net.URL;
import android.os.AsyncTask;
/**
 * 检测wifi是否需要登陆
 * 使用方式：
 * NetNeedLoginCheckUtil.needLoginNetworkCheck(new NeedLoginCallBack() {
 *  @Override
 *  public void needLogin(boolean needLogin) {
 *     if (needLogin) {
 *          wifi 需要登陆
 *     }
 *  }
 * });
 **/
 public class NetNeedLoginCheckUtil extends AsyncTask<Integer, Integer, Boolean>{
    NeedLoginCallBack callBack;
    public NetNeedLoginCheckUtil(NeedLoginCallBack callBack) {
        super();
        this.callBack = callBack;
    }    
    @Override
    protected Boolean doInBackground(Integer... params) {
        return isWifiSetPortal();
    }    
    @Override
    protected void onPostExecute(Boolean result) {
        if (callBack != null) {
            callBack.needLogin(result);
        }
    }
    private boolean isWifiSetPortal() {  
        final String mWalledGardenUrl = "http://connect.rom.miui.com/generate_204";  
        final int WALLED_GARDEN_SOCKET_TIMEOUT_MS = 10000;  
        HttpURLConnection urlConnection = null;  
        try {  
            URL url = new URL(mWalledGardenUrl);  
            urlConnection = (HttpURLConnection) url.openConnection();  
            urlConnection.setInstanceFollowRedirects(false);  
            urlConnection.setConnectTimeout(WALLED_GARDEN_SOCKET_TIMEOUT_MS);  
            urlConnection.setReadTimeout(WALLED_GARDEN_SOCKET_TIMEOUT_MS);  
            urlConnection.setUseCaches(false);  
            urlConnection.getInputStream();  
            return urlConnection.getResponseCode() != 204;  
        } catch (IOException e) {  
            return false;  
        } finally {  
            if (urlConnection != null) {  
                urlConnection.disconnect();  
            }  
        }  
    }  

    public static void needLoginNetworkCheck(NeedLoginCallBack callBack) {
        new NetNeedLoginCheckUtil(callBack).execute();
    }    

    public interface NeedLoginCallBack{
       void needLogin(boolean needLogin);
    }
}
```

文／李科吐温（简书作者）
原文链接：http://www.jianshu.com/p/3187c677bca3
著作权归作者所有，转载请联系作者获得授权，并标注“简书作者”。
