---
title: cross domain
author: jyy
date: 2021-02-25 11:10:00 +0800
# categories: [Blogging, Tutorial]
tags: [js]
pin: true
---

## 跨域：是指浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器对JavaScript实施的安全限制。
> 同源策略：是指协议，域名，端口都要相同，其中有一个不同都会产生跨域

## 如何解决跨域

### 1.jsonp

> 1.主要就是利用了 script 标签的src没有跨域限制来完成的 2.只能发送get请求 3.不安全，xss攻击
> 原理：前端自定义一个callback函数，向后端发送一个带有callback参数的get请求，服务端返回一个callback的执行函数并且带有参数。前端就会执行这个callback函数并且拿到其中的参数。

```js
//原生js实现
<script>
    var script = document.createElement("script");
    script.src = "http://localhost:3000/login?user=jyy&callback=show"
    function show(res){
        console.log(res)
    }
    document.body.appendChild(script)
</script>
```

```js
//jsonp实现
function jsonp({url, params,callback}){
    return new Promise((resolve,reject) => {
        let script = document.createElement("script")
        window[callback] = function(data){
            resolve(data)
            document.body.removeChild(script)
        }
        params = { ...params, callback }
        let attrs = []
        for (let key in params) {
            attrs.push(`${key}=${params[key]}`)
        }
        script.src = `${url}?${attrs.join("&")}`
        document.body.appendChild(script)
    })
}
jsonp({
    url: "http://localhost:3000/login",
    params: {user: "jyy"},
    callback: show
}).then(res=>{
    console.log(res)
})
```

### 2.cors(跨域资源共享)
> 它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制

cors主要是服务端需要做一些是否同意接受请求的处理。
- Access-Control-Allow-Origin
它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求

- Access-Control-Request-Method
请求方法是PUT或者DELETE时，需要设置

- Access-Control-Allow-Headers
该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段

- Access-Control-Allow-Credentials
表示是否允许发送Cookie。默认情况下，跨域时Cookie不包括在CORS请求之中

- Access-Control-Max-Age

预检请求：请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求，预检请求的请求方法是OPTIONS，表示这个请求是用来询问的

用来指定本次预检请求的有效期，单位为秒，在此期间，不用发出另一条预检请求

- Access-Control-Expose-Headers

CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定

### 3.postMessage
主要解决以下方面的问题：
- 页面和其打开的新窗口的数据传递
- 多窗口之间消息传递
- 页面与嵌套的iframe消息传递

```js
//a.html(http://localhost:3000/a.html)
<iframe src="http://localhost:4000/b.html" id="iframe" onload="load()"></iframe>
<script>
    function load() {
        let frame = document.getElementById("iframe")
        frame.contentWindow.postMessage("zhuzhu", "http://localhost:4000")
        window.onmessage = function (e) {
            console.log(e.data)
        }
    }
</script>
```

```js
//b.html(http://localhost:4000/b.html)
<script>
    window.onmessage=function(e){
        console.(e)
        e.source.postMessage("jyy","http://localhost:3000")
    }
</script>
```

### 4.window.name + iframe

> 浏览器窗口有window.name属性。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它

```js
//a和b页面同域 http://localhost:3000
//c页面 http://localhost:4000

//a页面
//把iframe地址修改为和a同域的b地址，这样a就能拿到c页面的window.name 
<script> 
    let tag = true
    function load() {
        if (tag) {
            let iframe = document.getElementById("iframe")
            iframe.src = "http://localhost:3000/b.html"
            tag = false
        } else {
            console.log(iframe.contentWindow.name)
        }
    }
</script>

//b页面
//空页面

//c页面
<script>
    window.name="jyyyyy"
</script>
```

### 5.window.hash + iframe

```js
//a和b页面同域 http://localhost:3000
//c页面 http://localhost:4000
//a页面
<iframe src="http://localhost:4000/c.html#zhuzhu" id="iframe" onload="load()">

    </iframe>
    <script>
    
        function load(){
            let iframe = document.getElementById("iframe")
        }
        window.onhashchange =function(){
            console.log(location.hash)
        }
      
    </script>
```
```js
//c页面
<iframe src="http://localhost:3000/b.html#jyyyy" id="iframe">

```

```js
//b页面
window.parent.parent.location.hash = location.hash
//window.parent.parent代表的是a页面
```


### 6.window.domain + iframe
> 此方案仅限主域相同，子域不同的跨域应用场景.两个页面都通过js强制设置document.domain为基础主域，就实现了同域

```js
//a页面 http://domain.com
<iframe src="http://b.domain.com/b.html" id="iframe">
<script>
    document.domain = 'domain.com';
    var user = 'admin';
</script>

```
```js
//b页面
<script>
    document.domain = 'domain.com';
    console.log(window.parent)
</script>
```