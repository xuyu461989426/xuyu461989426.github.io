---
layout: post
title:  "android 屏幕尺寸信息"
date:   2017-09-25 09:28:00 +0800
categories: ketan
cover: 'http://upload-images.jianshu.io/upload_images/555358-95bfa1e21a76e675.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240'
tags:  code 
---

 [TOC]

#### 获取android设备信息


##### 设备唯一识别号


```java
  public void getDeviceId(){
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.O) {
            TelephonyManager telecomManager = (TelephonyManager) getSystemService(TELEPHONY_SERVICE);
            telecomManager.getDeviceId();
        }
    }
    
    @RequiresApi(api = Build.VERSION_CODES.O)
    public void getDeviceIme(String tips) {
        TelephonyManager telecomManager = (TelephonyManager) getSystemService(TELEPHONY_SERVICE);
        telecomManager.getImei();
    }
```

##### 获取设备屏幕宽高

```java
    /**
     * 获取宽度 px
     * @param context
     * @return
     */
    public  static  int getDeviceWidth(Context context) {
        return  context.getResources().getDisplayMetrics().widthPixels;
    }

    /**
     * 获取高度 px
     * @param context
     * @return
     */

    public  static  int getDeviceheight (Context context) {
        return  context.getResources().getDisplayMetrics().heightPixels;
    }
    
    
    DisplayMetrics 的一些常见属性
    

```
 
 
 ![29a1f37229d3b6b215bf0421803ea471](Android 之获取硬件信息.resources/6536BB11-9969-4FD3-A749-0B7BF9555BC9.png)
 

首先我们要尺寸有个敏度度

1米三尺，一尺10寸 ,一寸3.3333厘米

一英寸=0.762寸=2.54 厘米

一般手掌7寸左右，18厘米，这个可以作为距离参照物。


我们再来了解屏幕的一些概念

* 像素  ：构成影像的最小单位，不可分割。记住像素有大小之分，演唱会后面屏幕在走进的时候很明显看到像素块。（px）
* 像素密度 ：对角线一英寸距离单位上有多少个像素。(dpi:dots per Inch)（印刷设备，鼠标,每移动一寸，界面跑过多少信息）
* 像素密度： ppi （屏幕显示）
* dp (density-independent pixel 一种无关像素密度的单位)
* sp（scale-independentpixel）


```java
为什么说dp是一种无关像素的单位 ：其实这个概念很混淆，我们获得dp的计算方式，是以一款固定大小尺寸手机为基准的  系统密度为160dpi的中密度手机屏幕为基准屏幕，即320×480的手机屏幕。在这个屏幕中，1dp=1px

为什么要以这个尺寸为基准，那是因为是为了解决一个现实的问题
当以Px为显示单位的时候，如果同样是5寸的手机，一个正好1080*1920 的手机 显示一个 1080*1920的图片是正好显示的，但是如果5寸手机720*1080显示图片只能显示一部分，而如果一部 2072*4659（假设有的话）5寸手机，也会显示完全，但是只占用屏幕一半，同样一个5寸大小，对于当前的手机，一般来说都是显示同样大小，所以dp就是为了解决这个问题，让同样尺寸大小的手机，不同分辨率下，显示的图片大小在视觉上趋于一样的。

如何做呢？？就是dp,dp就是物理大小上的视觉单位（我定义的），（因为在android中大家经常用dp换算px常常，因为这个思维感到困扰）

视觉单位：是由尺寸和分辨率共同影响的。（视觉就是在人眼观看舒服的，这个舒服度就在在基准手机上观看的舒服度，


px=dp*(dpi/160)
为什么除以160 是因为分辨率都是以160为比例的

px/dpi = dp/160
 因为px是像素块，而dpi是一英寸有多少个像素块  所以px/dpi是像素块大小
 这个公式就是为了像素块。
 其实这个公式可以简化为px/dp =dpi 像素密度就是我们的视觉大小， px为显示单位,dp为反补因素，
 上面就是px和dp转换大小
 
  px/dpi 就是像素大小和像素密度的比例
  比例固定 = dp/160
  
  （总结一句话，就是为了统一视觉距离而出来了与像素无关的dp大小单位，其值是固定的480*320 密度是160点的设备上）
  
   如果按照像素来显示 肯定不统一，但是按照dp来显示肯定统一。因为像素大小变化的，距离是根据dp倒推像素多少
 
```
 1、设计稿 转化单位，为什么要除以0.75，或者除以2
 
 
 
 
 

| 分辨率 |  | 像素大小比例 |分辨率比例160|分辨率|
| --- | --- | --- | --- |---|---|
| ldpi  |  | 0.75 |  1.5|240||
| mdpi |  | 1 | 2 |320||
| hdpi |  | 1.5 |  3|480||
| xhdpi |  | 2 | 4.5 |720|
| xxhdpi |  | 3 | 6.75 |1080|

```java
     public static final int DENSITY_LOW = 120;
     public static final int DENSITY_MEDIUM = 160;
     public static final int DENSITY_TV = 213;
     public static final int DENSITY_HIGH = 240;
     public static final int DENSITY_XHIGH = 320;
     public static final int DENSITY_400 = 400;
     public static final int DENSITY_XXHIGH = 480;
     public static final int DENSITY_XXXHIGH = 640;
     public static final int DENSITY_DEFAULT = DENSITY_MEDIUM;
     
     这个android 官方文档中推荐dpi 一般来说，人眼在300多dpi已经舒适了，所以这个尺寸就是UI 最佳设计大小，可以由270（1080*720） 推测出手机稿高度，4英寸= 2.54*5 =12.16cm 这个都之前小手机
     现在多2341      400推出一般高度 5.8 = 14.87
     所以现在多为320-410密度浮动。
     
     
```

#### 默认基准分辨率
 
 我一直不知道默认values-下的分辨率到底是哪个分辨率，其实这个分辨不是那个分辨，而是看你放的是那个分辨率，放720的就是720的分辨率，但你如果你适配480的手机，默认这个分辨率的手机也会加载这个数值，就会导致图形变得很大，但是一般设计稿都是720，并且你常见的手机都是1080或者720，这就导致你一般看不到差别。
 
 所以最好还是适配下
 查看下美团的适配方案。
 

[屏幕分辨率参考](https://blog.csdn.net/ttkatrina/article/details/50623043)


##### 为什么要适配
 因为默认加载的都是values 下的dimen ，这个只能适配一个分辨率的机型，其他机型如果没有适配可能布局不合适，默认参考基础一般都是主流机型。
 [android对不同dpi如何选择dimen](https://blog.csdn.net/fire_tray/article/details/49967247)


*photoShop 栅格处理 将高分辨率 丢失精度转换为低分辨率。


#### DisplayMetrics 属性解析

```java
density  :逻辑密度比例
densityDpi ：像素密度
DisplayMetrics.DENSITY  像素密度

px sp dp 相互转换。

public static int px2dip(Context context, float pxValue) {
        final float scale = context.getResources().getDisplayMetrics().density;
        return (int) (pxValue / scale + 0.5f);
    }


    public static int px2sp(Context context, float pxValue) {
        float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int) (pxValue / fontScale + 0.5f);
    }


    public static int sp2px(Context context, float spValue) {
        float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
        return (int) (spValue * fontScale + 0.5f);
    }
    

float 类型的 4.1 和4.9 强转成int类型后，会失去精准度变成 int类型的4， 而如果咱们想四舍五入的话，把他们都加上0.5f，这样转换出来的结果就是：

4.4 + 0.5 = 4.9 转为int 还是4，而4.5 + 0.5 = 5.0 转换成int后就是5，正好是四舍五入，这样就保证了咱们算出来的值相对精准。
```