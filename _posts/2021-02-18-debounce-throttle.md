---
title: 防抖和节流
author: jyy
date: 2021-01-18 15:20:00 +0800
# categories: [Blogging, Tutorial]
tags: [js]
pin: true
---

## 为什么需要防抖和节流
> 比如用户进行sroll和resize等行为的时候，会导致绑定的回调函数频繁执行（如果回调函数里大量操作dom，卡顿会更加严重），接着页面会不停的重新渲染，会大大加重浏览器的负担。

### 防抖
> 在规定时间阀值内，回调函数只执行一次，如果在规定时间内再次触发该事件，则清空之前的定时器，重写开启一个定时器，等到该行为的空闲时间大于规定时间则定时器执行并执行其中的回调函数。

#### 应用场景

- 调整浏览器窗口大小的onresize事件，只需要在用户最后一次拖动完成后一段事件内执行就行，不需要频繁的执行。
- 文本编辑器实时保存，当无任何更改操作一秒后进行保存
- 各种按钮的频繁点击会导致多次请求，也要防抖。

```js
<body>
<button id="btn">快速点击</button>
</body>
<script>

</script>
function logger(){
    console.log("logger")
}
function debounce(fn, wait, immediate){
    //immediate代表第一次是否立即执行
    let timeout
    return function(...args){
        clearTimeout(timeout)
        if(immediate){
            let callNow = !timeout
            if(callNow) fn.apply(this,args)
        }
        timeout = setTimeout(()=>{
            fn.apply(this,args)
            timeout = null
    },wait)
    }
}
btn.addEventListener("click", debounce(logger, 2000, true))
```

### 节流
> 在规定的时间阀值内，频繁的触发事件，则只有一次会执行。

#### 应用场景

- 监听滚动事件，获取位置或者请求数据等，需要节流。
- input 框实时搜索并发送请求（也可做防抖）

```js
function logger(){
    console.log("logger")
}
function throttle(fn, wait, options = {}){
    let timeout
    let previous = 0
    return function(...args){
        let now = Date.now()
        if(options.leading === false && !previous) previous = now
        let remaining = wait - (now - previous)
        if(remaining <= 0){
            if(timeout){
                clearTImeout(timeout)
                timout = null
            }
            fn.apply(this,args)
            previous = now
        }else if(!timeout && options.trailing !== false){
            timeout = setTimeout(() => {
                previous = options.leading === false ? 0 : Date.now();
                fn.apply(this,args)
                timout = null
            },remaining)
        }
    }
}
//默认第一次是执行的
btn.addEventListener("click", throttle(logger, 2000, {trailing: true,leading: false}))
//trailing: true表示最后一次会执行
//leading：false表示第一次不执行
```
### 总结

- 防抖是将多次行为转化为最后一次执行，节流是多次行为转化为每隔一段时间执行一次。
