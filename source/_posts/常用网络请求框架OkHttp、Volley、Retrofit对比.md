---
title: 常用网络请求框架OkHttp、Volley、Retrofit,android-async-http对比
date: 2017-03-01 19:52:28
tags: Android
---
1.android-async-http
    android-async-http 是基于HttpClient的，目前HttpClient已经被废弃.
    貌似android-async-http已经把一整套的HttpClient加入代码中,所以目前还有人在用.

2.OkHttp
    Android 开发中是可以直接使用现成的api进行网络请求的。是Square公司开源的针对Java和Android程序，封装的一个高性能http请求库，它的职责跟HttpUrlConnection 是一样的，支持同步、异步，而且 OkHttp 又封装了线程池，封装了数据转换，封装了参数使用、错误处理等，api使用起来更加方便。可以把它理解成是一个封装之后的类似HttpUrlConnection的东西，但是在使用的时候仍然需要自己再做一层封装，这样才能像使用一个框架一样更加顺手。
<!--more-->
3.Volley
   Volley是Google官方出的一套小而巧的异步请求库，该框架封装的扩展性很强，支持HttpClient、HttpUrlConnection， 甚至支持OkHttp，而且Volley里面也封装了ImageLoader，所以如果你愿意你甚至不需要使用图片加载框架，不过这块功能没有一些专门的图片加载框架强大，对于简单的需求可以使用，稍复杂点的需求还是需要用到专门的图片加载框架。Volley也有缺陷，比如不支持post大数据，所以不适合上传文件。不过Volley设计的初衷本身也就是为频繁的、数据量小的网络请求而生。

4.Retrofit
   Retrofit是Square公司出品的默认基于OkHttp封装的一套RESTful网络请求框架，RESTful是目前流行的一套api设计的风格， 并不是标准。Retrofit的封装可以说是很强大，里面涉及到一堆的设计模式,可以通过注解直接配置请求，可以使用不同的http客户端，虽然默认是用http ，可以使用不同Json Converter 来序列化数据，同时提供对RxJava的支持，使用Retrofit + OkHttp + RxJava + Dagger2 可以说是目前比较潮的一套框架，但是需要有比较高的门槛。

5.Volley VS OkHttp
  Volley的优势在于封装的更好，而使用OkHttp你需要有足够的能力再进行一次封装。而OkHttp的优势在于性能更高，因为 OkHttp基于NIO和Okio ，所以性能上要比 Volley更快。IO 和 NIO这两个都是Java中的概念，如果我从硬盘读取数据，第一种方式就是程序一直等，数据读完后才能继续操作这种是最简单的也叫阻塞式IO,还有一种是你读你的,程序接着往下执行，等数据处理完你再来通知我，然后再处理回调。而第二种就是 NIO 的方式，非阻塞式， 所以NIO当然要比IO的性能要好了,而 Okio是 Square 公司基于IO和NIO基础上做的一个更简单、高效处理数据流的一个库。理论上如果Volley和OkHttp对比的话，更倾向于使用 Volley，因为Volley内部同样支持使用OkHttp,这点OkHttp的性能优势就没了，  而且 Volley 本身封装的也更易用，扩展性更好些。

6.OkHttp VS Retrofit
   毫无疑问，Retrofit 默认是基于 OkHttp 而做的封装，这点来说没有可比性，肯定首选 Retrofit。

7.Volley VS Retrofit
  这两个库都做了不错的封装，但Retrofit解耦的更彻底,尤其Retrofit2.0出来，Jake对之前1.0设计不合理的地方做了大量重构， 职责更细分，而且Retrofit默认使用OkHttp,性能上也要比Volley占优势，再有如果你的项目如果采用了RxJava ，那更该使用  Retrofit 。所以这两个库相比，Retrofit更有优势，在能掌握两个框架的前提下该优先使用 Retrofit。但是Retrofit门槛要比Volley稍高些， 要理解他的原理，各种用法，想彻底搞明白还是需要花些功夫的，如果你对它一知半解，那还是建议在商业项目使用Volley吧。

文章来源 : http://blog.csdn.net/qq_33342248/article/details/53906842
