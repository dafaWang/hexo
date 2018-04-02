---
title: Hybrid app入门
date: 2018-04-02 16:21:20
tags: 
     - 混合开发
     - H5
     - android
categories: 
     - android
     
---
Hybrid APP指的是半原生半Web的混合类App。需要下载安装，看上去类似Native App，但只有很少的UI Web View，访问的内容是 Web 。

例如Store里的新闻类APP，视频类APP普遍采取的是Native的框架，Web的内容。

Hybrid App 极力去打造类似于Native App 的体验，但仍受限于技术，网速，等等很多因素。尚不完美。


<!-- more -->

##### 通过本文我们将了解到:
1.android 如何通过webView加载网页
2.android 如何调用加载的网页中的js方法
3.js代码中如何调用android里的方法
4.android中如何拦截js点击事件
　
##### 在学习之前首先要了解一丢丢的html和js的知识

#### 一  :  (如何通过webView加载网页)
webView是google提供的一个加载网页的官方控件
通过loadUrl()方法注入需要加载的网页
因为HTML文件的位置有可能在服务器,有可能在本地所以加载的url也不同
         
      例:
         mWebView.loadUrl("file:///android_asset/test.html");//加载本地assets目录下html文件
         mWebView.loadUrl("http://www.baidu.com");//  加载网页
如果你只是这样写的话,不出意外的,手机肯定就自动打开某个默认浏览器来加载网页了,可是我们希望的是通过我们手机内部的webView来加载:
这时我们就得通过
    
    mWebView.setWebViewClient()
方法来设置一个webView代理,可以理解为,我们要通过webView这个代理来加载网页了,不需要默认浏览器了.
在参数中我们new一个代理

    mWebView.setWabViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {               
             //见名知意     将要重写url的加载过程
                view.loadUrl(url);   //-->通过webView来加载url
                return true;
            }
        });
这样一来就可以通过webView来加载界面了

#### 二  : (android 如何调用加载的网页中的js方法)

   如果想让webView调用js中的方法,首先要让webView支持使用js
        
    WebSettings settings = mWebView.getSettings();
    settings.setJavaScriptEnabled(true);
设置完成之后就可以调用js中的方法了

###### 注意 : 
     UsingsetJavaScriptEnabled
    can introduce XSS vulnerabilities 
     into you application, review carefully. 
     Your code should not invokesetJavaScriptEnabled
    if you are 
     not sure that your app really requires JavaScript support.

这里说如果支持了js可能会遭到XSS攻击

###### 1.有返回值的调用
    mWebView.evaluateJavascript(String s,ValueCallback<T> callback)//-->通过这个方法调用js方法
这个方法中需要两个参数, s  为js方法调用的拼接,比如说我们要调用js中的
sum(a,b)方法来求a和b的和
s就应该写作 "sum(a,b)"-->其中的a和b可以通过别的方式获取

第二个参数就是js方法的回调,ValueCallback的泛型为方法返回值类型
###### 2.没有返回值的调用
如果说我们调用的js方法没有返回值,就不用这么麻烦了
可以直接调用

    mWebView.loadUrl("javascript:do()")
js中的代码是这样:

    <script>
          function do(){
          ******
           }
    </script>
###### 注意:方法名要一致


##### 练习 : 点击一个按钮,利用js中的方法求出两数之和并返回通过textView展示

    public void useJs(View view) {
        //使用js中的方法
        mWebView.evaluateJavascript("sum(30,50)", new ValueCallback<String>() {
            @Override
            public void onReceiveValue(String s) {
                tv.setText("JS代码中sum()方法计算30+50的结果为 : " + s);
            }
        });
    }

#### 三  :  (js代码中如何调用android里的方法)

app中的代码很简单
android4.2以后可以直接使用@JavascriptInterface注解来声明js方法了

    public class JsMethods {
        @JavascriptInterface
        public String show() {
            return "成功调用android中的方法";
        }
    }
这段代码中,show()方法直接返回了一串字符串
创建完JsMethods之后,我们需要给webView添加一个可以使用android代码的接口

    mWebView.addJavascriptInterface(new JsMethods (), JS_NAME);
    //      public static final String JS_NAME = "android";
这里JS_NAME是一个字符串常量,作为连接js和android的一个通道,
将需要交互的方法封装在JsMethods 实例化的对象中,添加到webView中供其使用.

js部分,就是通过JS_NAME这个通道来获取其中的方法 : 

    <script type="text/javascript">
		function android_method() {
			// body...
			var value = window.android.show();
			document.getElementById('p').innerHTML = value;
		}
	</script>
###### 注意 : window.android.show(); --> 这里的android是可变的,但是要与mWebView中设置的通道名儿一致,否则找不到需要的方法


#### 四 : (android中如何拦截js点击事件)
如果我们想获取网页中的一些链接,可以在开头提到的shouldOverrideUrlLoading()方法中进行拦截

比如在我们的test.html中有一个链接是跳转百度的,我们不想让它跳转,同时我们还要获取到他跳转的url

     mWebView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                if (url.contains("baidu")) {
                    Toast.makeText(MainActivity.this, "拦截界面跳转成功", Toast.LENGTH_SHORT).show();
                    tv.setText("拦截到的url为 : " + url);
                } else {
                    view.loadUrl(url);
                }
                return true;
            }
        });
#### 后话 :

关于webview的更多api:
[api](https://developer.android.google.cn/reference/android/webkit/WebView.html)

关于demo的完整代码:

 [完整代码](https://github.com/dafaWang/H5SimpleAPP)
