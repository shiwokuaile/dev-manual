## 微信小程序

[RequestTask | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/api/network/request/wx.request.html)

[小程序登录 | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/user-login/code2Session.html)

**1、uploadFile**

wx.request 不支持 formData 提交，需要特殊处理，可以参考：[wx.request 发起 formData 请求](https://developers.weixin.qq.com/community/develop/article/doc/0000cc0e5bc5d093c6f8be17254c13)

[UploadTask | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/api/network/upload/wx.uploadFile.html)

```javascript
    wx.uploadFile({
      url: config.apiHost + '/xxx/yyy',
      // 微信本地文件路径（结合 wx.chooseImage 使用）
      filePath: filePath,
      name: 'pic',
      // 其他数据
      formData: {
        'user': JSON.stringify(personal)
      },
      success(res) {
      }
    })
```

> 注意：
> 
> wx.uploadFile 方法不灵活，没法像其他客户端那样可以设置 formData 表单中字段对应内容的类型。
> 
> 对于 SpringBoot 后台，需要在接口通过 HttpServletRequest 获取表单中的参数数据。
