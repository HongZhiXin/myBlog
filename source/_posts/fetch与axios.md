---
title: fetch与axios
categories: JavaScript
date: 2020-03-25 22:16:26
top_img: http://img.netbian.com/file/2020/0322/0bed5d59f12720752019fc2b8ad48993.jpg
cover: https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=4005754747,4127172206&fm=26&gp=0.jpg
---

### 背景

fetch与axios都是promise出现后优秀的异步请求接口数据的方案，简洁的语法代替ajax十分舒服

### 优缺点

fetch是原生js支持的，不需要引入插件，但是对IE低版本不太友好。

axios除了需要引入以外，几乎没有缺点，本身是基于XMLHttpRequest原理的，所以是个web都能支持，除了浏览器端，node端也支持。

<!-- more -->
### 开始使用

定义需要的参数

```javascript
const URL = "https://free-api.heweather.net/s6/weather/now";//使用了天气free版api接口
const loc = "shenzhen";//接口定义的城市参数
const key = "******************"; //真实key值这里不显示
```

- fetch （get方法）

  ```javascript
  fetch(URL+'?'+`location=${loc}`+'&'+`key=${key}`).then(data=>
        data.json() 
       ).then(data=>{
         console.log(data,"fetch")
       })
  ```

  fetch()返回一个promise对象，然后该对象.then()中转为json数据格式并返回一个新的promise,新的promise对象.then()中拿到转为json格式的数据。

- fetch （post方法）

  由于使用的真实接口是get方法的，可以直接把参数拼接在地址上，这里的post方法使用的是网上的代码

  这里的关键是传入头部content-type，还有一个注意的地方则是，如果将config对象中的method填入'GET'会报错，提示GET/HEAD方法不能接收body参数。所以get方法只能通过拼接参数到url上进行调用

  ```javascript
  let postData = {a:'b'};
  fetch('http://data.xxx.com/Admin/Login/login', {
    method: 'POST',
    mode: 'cors',
    credentials: 'include',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    },
    body: JSON.stringify(postData)
  }).then(function(response) {
    console.log(response);
  });
  ```

- axios

  首先在node环境下使用命令npm install axios 安装到axios项目根目录下的node_modules中

  在项目中引用

  ```javascript
  import axios from "axios"
  ```

  三个参数继续沿用，调用如下

  ```javascript
  axios.request({
          url: URL,
          method: "get",
          params: {
            location: loc,
            key: key
          }
        })
        .then(data => {
          const newdata = data.data;
          //判断是否成功取得数据
          if (newdata.HeWeather6 && newdata.HeWeather6.length > 0) {
          //对data中的初始变量进行赋值，便于页面显示
            const weather = newdata.HeWeather6[0];
            this.weather = weather.now;
            this.lOcation = weather.basic;
            this.timer = weather.update;
          }
        });
  ```

  

### 总结

本人更倾向于axios，因为现在前后端分离的项目都会有node环境啦，又不需要考虑兼容性问题，真香！