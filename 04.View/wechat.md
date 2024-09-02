[RequestTask | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/api/network/request/wx.request.html)

[小程序登录 | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/user-login/code2Session.html)

## uploadFile

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



## 开发模式转换

1、将原生开发转为云开发模式

[微信小程序 - 非云开发项目转云开发项目_微信小程序最开始没有使用云开发,后期想用云开发-CSDN博客](https://blog.csdn.net/chengqige/article/details/117739685)

修改 project.config.json

```json
// 配置云函数根目录
 "cloudfunctionRoot": "cloudfunciton"
```

修改 app.js 文件

```javascript
  onLaunch: function () {
    if (!wx.cloud) {
      console.error('请使用 2.2.3 或以上的基础库以使用云能力');
    } else {
      wx.cloud.init({
        // env 参数说明：
        //   env 参数决定接下来小程序发起的云开发调用（wx.cloud.xxx）会默认请求到哪个云环境的资源
        //   此处请填入环境 ID, 环境 ID 可打开云控制台查看
        //   如不填则使用默认环境（第一个创建的环境）
        env: 'env-8g0qwlro691d64bb',
        traceUser: true,
      });
    }
    this.globalData = {};
  },
```

## 获取用户头像和昵称

可以选择微信头像，也可以重新设置，但是昵称需要自定义。参考：[头像昵称填写 | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/userProfile.html)

基础版本 2.27.3 以下，wx.getUserProfile 和 wx.getUserInfo 可以直接获取用户的微信头像和昵称。

[wx.getUserProfile(Object object) | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/user-info/wx.getUserProfile.html)

[wx.getUserInfo(Object object) | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/user-info/wx.getUserInfo.html)

基础库版本在 2.27.3 以上，wx.getUserProfile 和 wx.getUserInfo 获取不到用户头像和昵称，返回的是默认的头像和昵称。

[小程序登录、用户信息相关接口调整说明](https://developers.weixin.qq.com/community/develop/doc/000cacfa20ce88df04cb468bc52801)

[小程序用户头像昵称获取规则调整公告](https://developers.weixin.qq.com/community/develop/doc/00022c683e8a80b29bed2142b56c01)

在开发者工具可以指定基础库版本，但无法指定客户端的基础库版本

[用户端的基础库能指定吗？](https://developers.weixin.qq.com/community/develop/doc/0006628833080052dd91ce92464400?highLine=%25E5%25B0%258F%25E7%25A8%258B%25E5%25BA%258F%25E5%258F%2591%25E5%25B8%2583%25E7%25BA%25BF%25E4%25B8%258A%25E5%25A6%2582%25E4%25BD%2595%25E6%258C%2587%25E5%25AE%259A%25E5%259F%25BA%25E7%25A1%2580%25E7%2589%2588%25E6%259C%25AC%25E5%25BA%2593)

[基础库版本分布](https://developers.weixin.qq.com/miniprogram/dev/framework/client-lib/version.html)

## 小程序注册（企业主体）

[小程序注册指引](https://developers.weixin.qq.com/community/business/doc/000200772f81508894e94ec965180d)

[产品定位及功能介绍 | 微信开放文档](https://developers.weixin.qq.com/miniprogram/introduction/?t=19041716)

## 微信支付

[云开发如何接入微信支付实践](https://developers.weixin.qq.com/community/develop/article/doc/000ecc7c48034043169ab04f456813)

[小程序接入微信支付](https://developers.weixin.qq.com/community/business/doc/000ce044e28c38013f6904bc05180d)

[小程序里面如何实现用户提现到微信余额当中](https://developers.weixin.qq.com/community/develop/doc/00028c16d406b82a59ee9449251400?_at=1674832590388)

[产品介绍-小程序支付 | 微信支付商户平台文档中心](https://pay.weixin.qq.com/wiki/doc/apiv3/open/pay/chapter2_8_0.shtml)

[小程序支付文档、](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=7_10&index=1)

## 微信优惠券

[一文了解微信优惠券产品（卡券、代金券、商家券）](https://developers.weixin.qq.com/community/develop/article/doc/000460b5934fd0f7f1eb902a251013)

[产品介绍-商家券 | 微信支付商户平台文档中心](https://pay.weixin.qq.com/wiki/doc/apiv3/open/pay/chapter5_2_1.shtml)

[商家券接口-微信支付-开发者文档](https://pay.weixin.qq.com/wiki/doc/apiv3/apis/chapter9_2_1.shtml)

## 微信地图

wx.chooseLocation 实现地图选址，不收费，但需要开通，并且在隐私声明中注明用途。

[wx.chooseLocation(Object object) | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/api/location/wx.chooseLocation.html)

[商用小程序使用wx.chooseLocation、getLocation等位置api需要商业授权吗？](https://developers.weixin.qq.com/community/develop/doc/00028abb1a4730e68710115846b400)

[选择地理位置接口调整公告](https://developers.weixin.qq.com/community/develop/doc/0006e45df2cac030e6edf367c56001)

[微信小程序API 之 位置API wx.getLocation(OBJECT)、wx.chooseLocation(OBJECT)、wx.openLocation(OBJECT)_小程序 wx.getlocation-CSDN博客](https://blog.csdn.net/qq_37968920/article/details/82315755)

## 实名认证

1、通过城市服务校验，目前停止开放。

[城市服务实名信息校验接口说明【监管原因，暂停开放】](https://developers.weixin.qq.com/community/business/doc/000e06614ac74068f3d9237eb5440d?page=1)

2、微信人脸核身

人脸识别作身份验证，但是对小程序所属的目录具有一定的要求，例如：政务、交通等

[微信人脸核身接口能力](https://developers.weixin.qq.com/community/develop/doc/000442d352c1202bd498ecb105c00d?highline=%E4%BA%BA%E8%84%B8%E6%A0%B8%E8%BA%AB)

## 消息订阅

小程序消息订阅

[小程序订阅消息（用户通过弹窗订阅）开发指南 | 微信开放文档](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/subscribe-message.html#%E8%AE%A2%E9%98%85%E6%B6%88%E6%81%AF%E6%B7%BB%E5%8A%A0%E6%8F%90%E9%86%92)

[小程序模板消息能力调整通知](https://developers.weixin.qq.com/community/develop/doc/00008a8a7d8310b6bf4975b635a401?blockType=1)

小程序通过公众号发送消息

[【已成功】只用云开发，如何发送公众号模板消息？](https://developers.weixin.qq.com/community/develop/article/doc/000442446203b0840660e18c36b813)

[模板消息 | 微信开放文档](https://developers.weixin.qq.com/doc/offiaccount/Message_Management/Template_Message_Interface.html#%E5%8F%91%E9%80%81%E6%A8%A1%E6%9D%BF%E6%B6%88%E6%81%AF)

[only_cloud_send_template: 小程序只用云开发发送公众号模板消息](https://gitee.com/mosmos_admin/only_cloud_send_template)
