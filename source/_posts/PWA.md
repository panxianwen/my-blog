---
title: PWA
author: 文子
tags:
  - PWA
  - JS
categories:
  - Web技术
abbrlink: 67dc4d37
---
### PWA简单入门及一些使用场景

#### 简介

PWA（Progressive Web Apps，渐进式Web应用）运用现代的 Web API 以及传统的渐进式增强策略来创建跨平台 Web 应用程序，使其具有与原生应用相同的用户体验优势。

使用浏览器缓存技术可离线使用的Web应用，区别于Native App与Hybrid App (Ionic、ReactNative、Weex等)。

#### 使用场景

* 支持离线使用
* 用户的网络环境为弱网环境（如：应急管理场景）
* 首屏优化
* 数据量较大使用频率较高的单页面

#### PWA使用到的技术

* Web App Manifest (生成桌面图标)
* Service Worker (离线访问)
* Push API & Notification API (服务端消息推送)
* App Shell & App Skeleton (骨架屏)



#### 一. Web App Manifest

将Web应用的Icon添加到桌面，名称与图标可自定义

Web应用程序清单

```html
<link rel="manifest" href="/manifest.json"> 
```

manifest.json

```json
{
  "name": "MyWebApp", //应用名称
  "short_name": "PWA应用示例", //桌面应用的名称
  "start_url": ".", //指定用户从设备启动应用程序时加载的URL
  "display": "standalone", //(fullscreen、standalone、minmal-ui、browser)
  "background_color": "#fff",//启动画面颜色
  "description": "A simply readable Hacker News app.",
  "icons": [{ //设置桌面图片 ionc图标
    "src": "images/touch/homescreen48.png",
    "sizes": "48x48",
    "type": "image/png"
  }, {
    "src": "images/touch/homescreen72.png",
    "sizes": "72x72",
    "type": "image/png"
  }, {
    "src": "images/touch/homescreen96.png",
    "sizes": "96x96",
    "type": "image/png"
  }, {
    "src": "images/touch/homescreen144.png",
    "sizes": "144x144",
    "type": "image/png"
  }, {
    "src": "images/touch/homescreen168.png",
    "sizes": "168x168",
    "type": "image/png"
  }, {
    "src": "images/touch/homescreen192.png",
    "sizes": "192x192",
    "type": "image/png"
  }],
  "theme_color": "#aaa" // 状态栏的颜色
}
```

针对IOS的配置

```html
<link rel="apple-touch-icon" href="icon.png">
<meta name="apple-mobile-web-app-title" content="PWA应用示例">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="theme-color" content="#000000">
```



用Android的chrome浏览器访问，然后通过设置-添加到桌面功能，就可以将你的应用添加到桌面了。
iOS不支持添加到桌面功能，用iOS的safari浏览器访问应用，然后通过底部的分享按钮-添加到主屏幕。

添加到桌面后，PWA 并不是一个快捷方式，而是能够在系统中作为一个独立的 App 存在的，用户可以设置它的权限，清除它的缓存，就和 Native App 一样。



#### 二. Service Worker

**Service Worker的特点**

* 一个特殊的 worker 线程，独立于当前网页主线程，有自己的执行上下文
* 自动休眠，不会随浏览器关闭而失效 (需要手动卸载)
* 可拦截并代理请求和处理返回，可以操作本地缓存，如 CacheStorage，IndexedDB 等
* 离线缓存内容开发者可控，可以控制动态请求，第三方静态资源等
* 不能直接访问 / 操作dom
* 所有api都基于Promise实现
* 能接受服务器推送的离线消息
* 必须在 HTTPS / Localhost环境下才能工作

**Service Worker的浏览器支持**

Chrome V40 版本就已经支持 Service Worker，Apple 方面从 MacOS Safari 11.1 和 iOS Safari 11.3 开始全面支持，IE Edge 从 17 版本开始全面支持。

**生命周期**

* 安装(installing)：在主线程成功注册 Service Worker 之后，开始下载并解析执行的过程中，触发 install 事件回调，这里可以指定一些静态资源进行离线缓存。
* 安装后(installed)：install 事件回调成功执行，Service Worker 已经完成了安装，如果 install 事件回调执行失败，则生命周期进入 Error 终结状态，终止生命周期。
* 激活(activating)：这个状态表示允许当前的 worker 完成安装，并且清除了其他的 worker 以及关联缓存的旧缓存资源，等待新的 Service Worker 线程被激活。
* 激活后(activated)：该状态下会处理 activate 事件回调 (提供了更新缓存策略的机会)。并可以处理功能性的事件 fetch (请求)、sync (后台同步)、push (推送)等。
* 废弃状态(redundant)：这个状态表示 Terminated 终结状态。激活后 Service Worker 被 unregister 或者有新的 Service Worker 版本更新，则当前 Service Worker 生命周期完结。

![avatar](./ServiceWorker.png)

**生命周期中重要的几个方法**

* install事件回调中的两个方法
  * **event.waitUntil()** ：传入一个 Promise 为参数，等到该 Promise 为 resolve 状态为止。
  * **self.skipWaiting()**：表示强制当前处在 waiting 状态的 Service Worker 进入 activate 状态
* activate事件回调中的两个方法
  * **event.waitUntil()** ：传入一个 Promise 为参数，等到该 Promise 为 resolve 状态为止。
  * **self.clients.claim()**：在 activate 事件回调中执行该方法表示取得页面的控制权, 这样之后打开页面都会使用版本更新的缓存。旧的 Service Worker 脚本不再控制着页面，之后会被停止。



###### 主要步骤

###### 1. 注册serviceWorker

 首页面index.html

```html
<script>
  if('serviceWorker' in navigator) {
    await navigator.serviceWorker.register('sw.js', {scope: '/'})
  }
</script>  
```

###### 2. 注册监听函数

 注册serviceWorker使用的sw.js中，其中self指当前的浏览器进程

```js
//Service Worker
console.log('Service Worker 注册成功')

self.addEventListener('install', ()=>{
    //安装回调
    console.log('Service Worker 安装成功')
})

self.addEventListener('activate', ()=>{
    //激活回调
    console.log('Service Worker 激活成功');
})

self.addEventListener('fetch', (event)=>{
    console.log('Service Worker 抓取请示成功：' + event.request.url);
})
```



###### 3. 缓存静态资源

安装时按缓存列表进行缓存处理

```js
//添加缓存
const version = 1
const CACHE_NAME = 'Version' + version
const CACHE_LIST = [
    '/',
    'index.html',
    'imgs/snake.png'
]
async function preCache() {
    let cache = await caches.open(CACHE_NAME)
    await cache.addAll(CACHE_LIST)
    await self.skipWaiting()
} 
self.addEventListener('install', (event)=>{
    //安装回调
    console.log('Service Worker 安装成功')
    event.waitUntil(preCache()) //跳过等待，直接激活
})
```

激活后，清除无用的缓存数据

```js
//清除无用缓存
async function clearCache() {
    self.clients.claim() // 让serviceWorker拥有控制权
    let keys = await caches.keys()
    await Promise.all(keys.map(key=>{
        if(key !== CACHE_NAME ) {
            return caches.delete(key)
        }
    }))
}
self.addEventListener('activate', (event)=>{
    //激活回调
    console.log('Service Worker 激活成功');
    event.waitUntil(clearCache())
})
```



###### 4. 离线使用缓存

```js
self.addEventListener('fetch', (event)=>{
    console.log('Service Worker 抓取请示成功：' + event.request.url)
    let url = new URL(event.request.url)
    console.log(url.origin, self.origin)
    if(url.origin !== self.origin) { // 静态资源走浏览器缓存
        return
    }
    event.respondWith(
        fetch(event.request).catch(err=>{
            console.log(event.request)
            return caches.match(event.request)
        })
    )
})
```



###### 5. 缓存策略

* network first（网络优先）
  * 该策略会优先尝试发送网络请求获取资源，在资源获取成功的同时会复制一份资源缓存到本地，当网络请求失败时再尝试从本地缓存中读取缓存资源。
  * 一般适用于对请求的实时性和稳定性有要求的情况
* cache first（缓存优先）
  * 该策略会优先从本地缓存读取资源，读取失败后再发起网络请求，成功获得网络请求响应结果时会将该结果缓存到本地。
  * 对于实时性要求不太高的资源，可以使用该策略提高加载速度。
* network only（仅网络）
  * 仅通过发送正常的网络请求获取资源，并将请求响应结果直接返回。
  * 适用于对实时性要求非常高的资源，或者是无需进行缓存的资源。比如验证码图片、统计数据请求等等
* cache only（仅缓存）
  * 仅从缓存中读取资源。
  * 这个策略一般需要配合预缓存方案使用
* StaleWhileRevalidate（从缓存取，用网络数据更新缓存）
  * 该策略跟 cache first 策略比较类似，都是优先返回本地缓存的资源。不同的地方在于，该策略无论在缓存读取是否成功的时候都会发送网络请求更新本地缓存。
  * 优点：在保证资源请求响应速度的同时，还能够保证缓存中的资源一直保持一个比较新的状态
  * 缺点：每次请求资源的时候，都会发起网络请求占用用户的网络带宽



###### **Workbox**

Workbox 是 Google Chrome 团队推出的一套 PWA 的解决方案，这套解决方案当中包含了核心库和构建工具，因此我们可以利用 Workbox 实现 Service Worker 的快速开发。



#### 三. 总结

* 了解了PWA使用到的技术栈，一些特性和生命周期
* 如何配置一个简单的PWA应用，及对已有项目的改造是渐进式的
* 应用缓存的常见的几种策略及实例说明



*相关链接*

[vue/cli-plugin-pwa](https://github.com/vuejs/vue-docs-zh-cn/blob/master/vue-cli-plugin-pwa/README.md)

[workbox](https://github.com/GoogleChrome/workbox)

[PWA应用实战](https://lavas-project.github.io/pwa-book/)

[PWA应用的改造](https://lzw.me/a/pwa-service-worker.html)

[PWA实例](https://www.cnblogs.com/chenyablog/p/7647378.html)

