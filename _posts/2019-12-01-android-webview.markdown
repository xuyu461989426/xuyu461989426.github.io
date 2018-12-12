---
layout: post
title:  "android WebView的一些使用总结"
date:   2017-09-26 09:28:00 +0800
categories: ketan
cover: 'https://ketanxu.com/assets/noteimg/img_webview.png'
tags:  code 
---

 [TOC]
#### 前言
在Android 中内嵌H5是一种很常见的事情，但是有时候我们会遇到很多问题，借来说说我遇到的几个小问题。




##### H5无法加载，网页显示不完全，或者显示不完整。

```java
 { //加载不显示页面处理方案
            if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.LOLLIPOP) {
                webView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
            }
            webView.getSettings().setDomStorageEnabled(true);
        }
```

在Android 5.0开始，WebView已经严格限制了加载不安全来源内容，有时候当前WebView加载很多不安全外部链接，WebView是不支持的，也就是显示空白页面或者显示不完整页面，一部分原因是H5页面内用引用了第三方东西，但是当我们无法控制来源内容的时候，我们可以加上
setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW)
这个表明，运行多种内容加载，包括当前安全源加载了不安全的内容。

当然这个一般是偶然的例外，如果H5端是自己公司的的并且能够控制内容，一般是可以内部解决的，还有一种情况也可能导致无法加载完成，就是如果H5采用了Storage 缓存一些判断数据，如果WebView没有开始缓存策略（注意这个是运行H5操作WebView进行缓存，而不是WebView缓存H5页面），可能导致H5逻辑判断失败，这时候需要加上
setDomStorageEnabled(true)



##### H5重定向导致goback无限循环

 为了精确控制WeView的H5前进和后退，我们需要制定自己的后退操作点击，一般是返回上一步，外加完全退出。
 
 其中完全退出，和返回上一步在正常访问都可以快速实现，借用 finish 和 goback Android功能，但是当有重定向的问题的时候，在IOS中和浏览器起可以正常返回，但是在Android WebView就形成了无限循环
 
 1、首先我们了解下H5中几种重定向的方法
 
 ```html
    window.location.href
    window.history.back
    window.navigate
    self.location
    top.location
    header url
```
 可见并不是所以的重定向都会在Reponse中返回 window.location
 也就我们无法通过访问返回的结果来判断是不是该URl是重新定向。 这种思路只能放弃
 
 
 2、采用WebView的一些监听方法
 
 ```
  监听WebView加载
  webView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                WebView.HitTestResult hit = webView.getHitTestResult();
                int hitType = hit == null ? WebView.HitTestResult.UNKNOWN_TYPE : hit.getType();
                if (hitType != WebView.HitTestResult.UNKNOWN_TYPE) {
                    webView.loadUrl(url);
                    return true;
                } else {
                    //重定向时hitType为0 ,执行默认的操作
                    return false;
                }
            }

}
这个是WebViewClient中的一个继承的方法
shouldOverrideUrlLoading  这个方法 在页面进行重定向会进行调用，但是这个并不是仅仅当重定向时候调用，当跳转或者加载非当前页面内容的时候（就是离开当前页面的很多操作），会加载JS或者html等很多链接，我们无法记录当前页面是否是重定向页面，并且也无法确定前面第几个步骤是重定向开始页面。
 ```
 
 再翻找一下WebView的一些功能，我们可以看到WebView 提供了BackForWardList（）
 这个就是控制WebView 在前进和后退的堆栈。我尝试下打印堆栈中的链接
 
 假如：A,B,C(其中A是原始页面，B是点击链接，C就是重载界面)
 当我点击goback 程序会执行
 C页面finish ->B 重载-> C结果堆栈内部打印还是 A,B,C 这样无限循环，虽然我们明明知道B链接重定向，最终加载C。但是我们还是无法解决一个问题，无法确定多个跳转那个是重定型链接。
 
 我尝试打印 shouldOverrideUrlLoading url链接
 当我点击goback 
 
 url为。B链接 -> 1，2，3，4（这里是指多个JS或者Html的跳转）->然后跳转到C链接,
 
 这个有规律吗？？这个真的没有规律吗？？这个肯定有规律。
 1、WebView 堆栈只会记录，重定向开始链接和重定向结束链接。
 2、当我们点击goback 最终WebView会再次加载C链接，并且WebView History堆栈还是会记录B和C。
 3、也就是说我们只要 点击goback B和C被清出堆栈，而又再次进入堆栈。
 
 
 
 
```mermaid
graph TD
A[A] -->|跳转| B(B)
C -->|goback| B[B]
B-->D{shouldOverrideUrlLoading}
D-->C[C]
```

 
也就是我们只要记录上次最终加载的Html然后载shouldOverrideUrlLoading进行判断就行了，如果上次加载最终链接是当前将要加载的链接，那么我们不进行架子啊并且，goBackOrForward（）进行指定step出入堆栈，让重定向开始的B也结束掉，回到A即可，相当于我们也进行了重定向。这样就到达我们破解无限循环的目的了。
