---
layout: post
title: 微信小程序登录token问题==封装在request请求中
date: 2022-07-15
category:
    前端小技
tags:
    微信小程序
---
小程序中关于登录流程有这样一个问题：  
一般的小程序开发都是在app.js的onLaunch中，通过wx.login调用一次后端接口，拿到token、用户信息等数据。   
而在进入的首页中，以pages/index/index页面为例。 
一般情况下，在进入一个页面也需要调用接口获取页面数据，但这个页面的数据需要由wx.login调用接口返回的token，这个时候，由于app.js中的接口还未返回，所以会出现当前页面调用接口不成功的问题。   
于是，针对这个问题，经过小一番代码测试，将登陆接口封装在了API请求的request方法中，做个代码记录。
<!--more-->
代码示例如下：
```
// app.js
App({
  onLaunch() {
  }
})
```
app.js中不做任何操作

重点在于request的封装,request中唯一的依赖为一个常量配置文件,配置文件为环境的BaseApi和配置白名单(不需要token的接口)
```
// config.js
const envVersion = __wxConfig.envVersion
let Base_URL;
if (envVersion === 'develop') { //开发版
  Base_URL = 'https://blog.liugezhou.online'
} else if (envVersion === 'trial') { //体验版
  Base_URL = 'https://blog.liugezhou.online'
} else if (envVersion === 'release') { //生产版
  Base_URL = 'https://blog.liugezhou.online'
}
let whiteApi = ['/loginApi] //不需要token的白名单-**loginApi为登陆接口**

module.exports = {
  Base_URL,
  whiteApi
}
```

在以上简单工作做好以后，直接将最后的request封装：

```
// request.js
const { Base_URL, whiteApi } = require('./config');

const getUserInfo = () => {
  return new Promise(resolve => {
    wx.login({
      success: async res => {
        const useInfo = await new Promise((resolveUser) => {
          resolveUser(request('/loginApi', 'POST', {
            code: res.code
          }, true))
        });
        wx.setStorageSync('token', useInfo.data.token)
        resolve(useInfo.data)
      }
    })
  })
}

// 参数依次代表 api/请求方法/接口数据/contentType类型
const request = async (api, method, data, type) => {
  let token = wx.getStorageSync('token') || '';
  if (!token && !whiteApi.includes(api)) {
    const result = await new Promise(resolve => {
      resolve(getUserInfo())
    })
    token = result.token
  }
  let ContentType = type ? 'application/json' : 'application/x-www-form-urlencoded'
  let url = Base_URL + api
  return new Promise((resolve, reject) => {
    wx.request({
      url,
      method,
      data,
      header: {
        'Content-Type': ContentType,
        'Authorization': 'Bearer ' + token
      },
      success(res) {
        resolve(res.data)
      },
      fail(error) {
        reject(error)
      }
    })
  })
}
module.exports = {
  // 创建用户身份 
  apiDemo: (data) => { 
    return request(`/apiDemo`, 'POST', data)
  },
}
```

最后模拟下业务，apiDemo这个接口是需要有token才能访问的：
```
// pages/index/index
import { apiDemo } from '../../utils/request'

api().then(res=>{})
```

这样讲获取token才能访问接口的异步等待问题就可以得到良好的解决，主要的问题是在request请求中同步获取结果这里。