---
title: 防抖（debounce）和节流（throttle）
date: 2023-01-31 10:00:43
tags: 面试
categories: JavaScript
---

#### 防抖:


**是什么**：用户某些交互行为是高频触发的，但只需要触发一次就好了。
<!-- more -->
**例如**：某输入框咱们在输入后会去进行搜索，每次用户都输入了新的内容都去更新是不明智的，使用了防抖后，会判断用户在某段时间内持续输入后，只会执行最后一次。

**实现原理**：事件触发 -> 开启一个定时器 -> 规定时间内再触发则重新计时(重写定时器) -> 定时到则触发

**代码演示** ：
```javascript
// 防抖函数
function debounce(fn, delay = 200) {
  let timer = 0
  return function() {
    // 如果这个函数已经被触发了
    if(timer){
      clearTimeout(timer)
    }
    timer = setTimeout(() => {
      fn.apply(this, arguments); // 透传 this和参数
      timer = 0
    },delay)
  }
}
```

#### 节流:


**是什么**：同样也是在用户的高频交互行为中，像控制阀门一样，控制了长时间中某个时间段的执行一次。

**例如**：在拖拽、滚动这类长时间监听的事件中做监听处理某业务，使用节流会在某时间段内实现监听一次。

**实现原理**：事件触发 -> 执行操作 -> 关闭阀门一段时间 -> 关闭状态中无法执行操作 -> 到规定时间后打开阀门 -> 重复前面过程

**代码演示** ：
```javascript
// 节流函数
function throttle(fn, delay = 200) {
  let  timer = 0
  return function () {
    if(timer){
      return
    }
    timer = setTimeout(() =>{
      fn.apply(this, arguments); // 透传 this和参数
      timer = 0
    },delay)
  }
}
```

#### 区别
防抖是让你多次触发，只生效最后一次。适用于只需要一次触发生效的场景。
节流是让你的操作，每隔一段时间触发一次。适用于多次触发要多次生效的场景。