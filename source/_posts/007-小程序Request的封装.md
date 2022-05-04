---
title: 小程序Request的封装
date: 2019-05-28
updated: 2022-05-04
description: 最近做了两个小程序，业务相对比较简单，关于公益方面的，收获颇多，其中感觉在开发中明显提升开发效率以及减少代码量的就是request的封装，下面稍稍做个总结。
toc: true
categories:
- 前端小技
tags: 
- 小程序
---
<font color=blue>更新说明：对文章排版以及内容格式做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

> 最近做了两个小程序，业务相对比较简单，关于公益方面的，收获颇多，其中感觉在开发中明显提升开发效率以及减少代码量的就是request的封装，下面稍稍做个总结。
## 通用封装
>在utils文件夹下新建两个文件，config.js以及request.js,代码分别如下。
```
<!-- config.js-->
module.exports = {
  appid: "wxcXXXXXXXXXXXXXX349f",
  API_BASE_URL: 'https://liugezhou.github.io/',//暂时测试环境地址、上线需要修改
}
```
```
<!-- request-->
const CONFIG = require("./config.js")
const request = (url,method,data) => {
  let _url = CONFIG.API_BASE_URL+url
  return new Promise((resolve ,reject)=>{
    wx.request({
      url: _url,
        method:method,
      data:data,
      header: {
        'content-type': 'application/json' // 默认值
      },
      success(request) {
        resolve(request.data)
      },
      fail(error) {
        reject(error)
      },
      complete(aaa) {
        // 加载完成
      }
    })
  })
}
/**
  * 小程序的promise没有finally方法，自己扩展下
  */
Promise.prototype.finally = function (callback) {
  var Promise = this.constructor;
  return this.then(
    function (value) {
      Promise.resolve(callback()).then(
        function () {
          return value;
        }
      );
    },
    function (reason) {
      Promise.resolve(callback()).then(
        function () {
          throw reason;
        }
      );
    }
  );
}


  //所有接口定义在这里
module.exports = {
  request,
  //app.js登录
  login:(data) => {
    return request('WeChat/Login.aspx','POST',data)
  },
  //获取验证码
  getMessageCode: (mobile) => {
    return request('Donor/DonorPhoneCode.aspx?'+mobile,'POST')
  },
  ……
}
```
> 通过上面两个小文件我们就将request封装完毕，在业务层调用代码的时候只需要：
```
    const REQUEST = require('../../utils/request.js');
    …………………………………………
    var that =this
        REQUEST.login({
          tokenkey: res.code
        }).then(function(res){
             that.globalData.openId = res.data.tokenkey
             that.globalData.isBind = res.code
        }).catch(res => {//catch  fail在这里
            console.log('fail:',res); 
          }).finally(()=>{//finally在这里
           console.log('finally:', "结束"); 
          })
    ………………………………………
```
## 队列封装（不常用）
>一般情况下，上面的封装我们按着自己的需求稍微修改就应该可以方便使用。
接下来介绍的一种是接口队列化封装。因为在我们的开发需求中，每一次接口的请求需要上一个接口返回的两个数据，因此稍作整理如下：
```
    <!-- request.js -->
    const CONFIG = require("./config.js")
    let requestlist = [];
    let doing = false;
    const request = (link, data, back) => {
      var link = CONFIG.API_BASE_URL + link
      requestlist.push({ link: link, data: data, back: back })
      if (!doing) dorequest();
    }
    const dorequest = () => {

      doing = true
      let task = requestlist.shift();
      if (task) {
        wx.request({
          url: task.link,
          data: Object.assign({
             seq : wx.getStorageSync("seq") || '',
             token : wx.getStorageSync("token") || '',
             openid : wx.getStorageSync("openId") || ''}, task.data),
          method: 'POST',
          header: {
            'content-type': 'application/json' // 默认值
          },
          success: res => {
            wx.setStorageSync("seq", res.data.seq)
            wx.setStorageSync("token", res.data.token)
            task.back(res.data);
            dorequest();
          },
          fail:function(){
            wx.showToast({
              title: '服务器出现故障',
              image: '../../assets/icon/wrong.png',
              duration: 3000
            })
          },
          complete:function(){
          }
        });
      }
        else {
        doing = false;
      }
    
    };

    module.exports = {
      request,
      //获取用户openid
      getOpenid: (data,res) => {
        return request('get-wx-openid',data,res)
      },
      //获取验证码
      getLoginCode: (data,res) => {
        return request('send-phone-code', data, res)
      }
    }
```