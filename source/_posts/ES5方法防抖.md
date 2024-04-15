---
title: ES5方法防抖
date: 2021-04-23 14:51:53
tags: ["javascript", "es5","debounce"]
---

# 前言

发现 layui 的 layer 组件未对方法调用进行限制，如`confirm`中快速连续点击确定会执行多次方法，记录下解决方法。

# 防抖函数

```js
function debounce(func, wait, immediate) {
    var timeout;
    return function () {
        var context = this,
            args = arguments;
        var later = function () {
            timeout = null;
            if (!immediate) func.apply(context, args);
        };
        var callNow = immediate && !timeout;
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
        if (callNow) func.apply(context, args);
    };
}
```

# 调用

```js

function openPro() {
    layui.layer.confirm("确认？",function (index) {
        var params = {}
        console.log("确认一次")
        debouncedOpenProAjax(params)
}

var debouncedOpenProAjax = debounce(openProAjax, 2000, true);


function openProAjax(params) {
    console.log("openProAjax")
    $.ajax({...})
}
```

## 参考

-   [1] [ES5 Javascript Debounce Function](https://gist.github.com/Sidd27/daa8c600694ee99b62daabcba0af85cb)

-   [2] [How to properly debounce ajax requests](https://stackoverflow.com/a/23494429)
