---
title: flex布局实现div填充屏幕
date: 2021-03-10 15:41:52
tags: ["CSS", "flex"]
---

# 效果图

![flex](/images/20210310154237.png)

<!--more-->

# HTML

```html
<body>
    <div class="container">
        <div id="top-container" class="flex-box"></div>
        <div id="content-container" class="flex-box"></div>
        <div id="bottom-container" class="flex-box"></div>
    </div>
</body>
```

# CSS

---

需要将想要填充屏幕剩余空间的元素的所有父元素的`height`设定为`100%`，该元素`flex-grow`设为`1`

---

```css
html,
body {
    margin: 0;
    padding: 0;
    height: 100%;
}
.container {
    display: flex;
    flex-direction: column;
    height: 100%;
}
#top-container {
    background-color: antiquewhite;
    height: 4rem;
    display: flex;
    flex-direction: column;
    align-items: center;
}
#content-container {
    background-color: aquamarine;
    flex-grow: 1;
}
#bottom-container {
    background-color: burlywood;
    height: 4rem;
}
```
