---
title: ajax or axios
date: 2019-05-05 18:12:23
categories: 
- 网络协议
tags: 
- axios
---

## 起因
最近在全局修改项目的ajax请求，遇到些‘意料之外’的问题，so记录下~

## ajax
> 是对原生XHR封装，这里就不赘述了~

## axios   

### 发送请求
Example:
1.执行GET请求
```javascript
// 为给定 ID 的 user 创建请求
axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
   console.log(error);
  });

// 可选地，上面的请求可以这样做
axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });

```

2.执行POST请求
```javascript
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });

```

3.执行多个并发请求, 有点类似promise.all()
```javascript
function getUserAge(){
 return axios.get('/user/age');
}

function getUserPermissions(){
 return axios.get('/user/persimissions');
}

Axios.all([getUserAge(), getUserPermissions()]).then(
 axios.spread(function (acct, perms) {
    // 两个请求现在都执行完成
 }));
```


### 拦截器
在请求或响应被 then 或 catch 处理前拦截它们。
```javascript 
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
  });

```

> 如果想在稍后移除拦截器，可以这样：

```javascript
var myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

### 取消请求
业务场景： 当触发某一条件时停止请求
使用 cancel token 取消请求
#### 方法一：
可以使用 CancelToken.source 工厂方法创建 cancel token，像这样：
// 取消请求（message 参数是可选的）
source.cancel('Operation canceled by the user.');
```javascript 
var CancelToken = axios.CancelToken;
var source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function(thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // 处理错误
  }
});

// 取消请求（message 参数是可选的）
source.cancel('Operation canceled by the user.');
```

#### 方法二：
```javascript
var CancelToken = axios.CancelToken;
var cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // executor 函数接收一个 cancel 函数作为参数
    cancel = c;
  })
});

/ 取消请求
cancel();
```
## 修改请求方式遇到和ajax区别的点
1. axios默认的请求数据类型Content-Type: application/json, ajax不是。
2. 因为params是添加到url的请求字符串中的，用于get请求;
而data是添加到请求体（body）中的， 用于post请求。需要把Content-Type为application/x-www-form-urlencoded，

## 为啥要选用axios请求
官网介绍的几个优点：
* 从浏览器中创建 XMLHttpRequests
* 从 node.js 创建 http 请求
* 支持 Promise API
* 拦截请求和响应
* 转换请求数据和响应数据
* 取消请求
* 自动转换 JSON 数据
* 客户端支持防御 XSRF /CSRF https://www.jianshu.com/p/00fa457f6d3e