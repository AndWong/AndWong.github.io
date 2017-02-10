---
title: WebView 获取网页源码
date: 2017-02-10 19:59:27
tags: WebView 获取网页源码
---
Api-19以上获取源码的方式有修改，如下：
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    mWebView.evaluateJavascript(
            "(function() { return ('<html>'+document.getElementsByTagName('html')[0].innerHTML+'</html>'); })();",
            new ValueCallback<String>() {
                @Override
                public void onReceiveValue(String html) {
                    Log.d("HTML", "whd >>html:" + html);
                    mHtmlSource = html;
                }
            });
} else {
   view.loadUrl("javascript:window.local_obj.showSource('<head>'+" +
                "document.getElementsByTagName('html')[0].innerHTML+'</head>');");
}

Api-19以上获取源码的方式，如下：
Android： 在WebView中获取网页源码            

1. 使能javascript：
webView.getSettings().setJavaScriptEnabled(true);

2. 编写本地接口
final class InJavaScriptLocalObj {
    public void showSource(String html) {
        Log.d("HTML", html);
    }
}

3. 向网页暴露本地接口
webView.addJavascriptInterface(new InJavaScriptLocalObj(), "local_obj");

4. 编写自己的WebViewClient，并在onPageFinished中提取网页源码。
final class MyWebViewClient extends WebViewClient{   
    public boolean shouldOverrideUrlLoading(WebView view, String url) {    
        view.loadUrl(url);    
        return true;    
    }   
    public void onPageStarted(WebView view, String url, Bitmap favicon) {
        Log.d("WebView","onPageStarted"); window.imagelistner.getImage(this.src)
        super.onPageStarted(view, url, favicon);
    }     
    public void onPageFinished(WebView view, String url) {
        Log.d("WebView","onPageFinished ");
        view.loadUrl("javascript:window.local_obj.showSource('<head>'+" +
                "document.getElementsByTagName('html')[0].innerHTML+'</head>');");
        super.onPageFinished(view, url);
    }
}


组合在一起的代码为：
package com.hi.briancol.htmlsource;

import android.app.Activity;
import android.graphics.Bitmap;
import android.os.Bundle;
import android.util.Log;
import android.webkit.WebView;
import android.webkit.WebViewClient;

public class HtmlSource extends Activity {
    private WebView webView;
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        webView = (WebView)findViewById(R.id.webview);
        webView.getSettings().setJavaScriptEnabled(true);
        webView.addJavascriptInterface(new InJavaScriptLocalObj(), "local_obj");
        webView.setWebViewClient(new MyWebViewClient());
        webView.loadUrl("http://www.cnblogs.com/hibraincol/");
    }


   final class MyWebViewClient extends WebViewClient{   
        public boolean shouldOverrideUrlLoading(WebView view, String url) {    
            view.loadUrl(url);    
            return true;    
        }   
        public void onPageStarted(WebView view, String url, Bitmap favicon) {
            Log.d("WebView","onPageStarted");
            super.onPageStarted(view, url, favicon);
        }     
        public void onPageFinished(WebView view, String url) {
            Log.d("WebView","onPageFinished ");
            view.loadUrl("javascript:window.local_obj.showSource('<head>'+" +
                    "document.getElementsByTagName('html')[0].innerHTML+'</head>');");
            super.onPageFinished(view, url);
        }
    }

    final class InJavaScriptLocalObj {
        public void showSource(String html) {
            Log.d("HTML", html);
        }
    }
}


关键之处在于：
view.loadUrl("javascript:window.local_obj.showSource('<head>'+document.getElementsByTagName('html')[0].innerHTML+'</head>');");

运行，可以看到在showSource(String html)中打印了网页源码。
