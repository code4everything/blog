## 一次JSAPI微信支付引发的血案

> 真的很想吐槽一下微信支付的开发文档，很是混乱，硬是要让我们这些刚入门的小白折腾个一天半天才能让人看到那让人激动的支付窗口。就不能写一个完整的文档吗？

### 准备

公众号AppId，安全密钥，商户号，商户密钥以及证书，添加支付目录等

> 这里不再赘述，这部分内容网上有很多资料，你可以通过谷歌百度获得，因为这并不复杂

### 后端

这部分内容其实也是很简单的，使用 [weixin-java-tools](https://github.com/Wechat-Group/weixin-java-tools) 工具库即可轻松完成，支付部分的接口可参考 [DEMO](https://github.com/binarywang/weixin-java-pay-demo)

需要引入的Maven包：

``` xml
<dependency>
    <groupId>com.github.binarywang</groupId>
    <artifactId>weixin-java-pay</artifactId>
    <version>3.1.0</version>
</dependency>
<dependency>
    <groupId>com.github.binarywang</groupId>
    <artifactId>weixin-java-mp</artifactId>
    <version>3.1.0</version>
</dependency>
```

主要讲一下获取临时票据和调用凭证，这里的接口在配置微信支付的时候需要调用，参考代码：

``` java
@Autowired
private WxMpService wxMpService;

/**
 * 下面的数据可作为HTTP接口返回给前端
 */

/**
 * 临时票据
 */
wxMpService.getJsapiTicket();

/**
 * 生成签名算法，URL表示当前页面的URL
 */
wxMpService.createJsapiSignature(url);

/**
 * 获取调用凭证，code为前端获取到的code，方法返回accessToken和openId等数据
 */
wxMpService.oauth2getAccessToken(code);
```

> [获取微信网页授权](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842)

### 配置

在初始化网页时，我们需要调用 `HTTP` 接口生成签名算法，然后用返回的数据配置微信，并选择需要用到的接口列表

``` javascript
wx.config({
  debug: true,
  appId: data.appId,
  timestamp: data.timestamp,
  nonceStr: data.nonceStr,
  signature: data.singature,
  jsApiList: ['chooseWXPay']
})
```

> [所有JS接口列表](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)

然后获取微信网页授权（上面已经给出链接），需要传参重定向页面的 `URL` ，然后用获取到 `code` 调 `HTTP` 接口获取调用凭证（包括 `accessToken` 和 `openId` ）

### 支付

> 基本上坑都在这里

在发起支付前，我们需要先统一下单，然后会用到下单返回的 `prePayId` 弹出微信支付窗口

> 重点讲一下签名，需要按照 `ASCII` 码顺序对参数进行排序，最后将商户密钥即 `Key` 拼接到后面，建议使用MD5进行签名

``` javascript
// 用到MD5加密库
import md5 from 'js-md5'
```

1. **统一下单**

    需要传递的参数在文档里面都能找到，最重要的是签名
    
    ``` java
    // 生成随机字符串
    let nonce = Math.random().toString(36).substr(2)
    // 生成签名
    md5(`appid=${appId}&body=${body}&device_info=WEB&mch_id=${mchId}&nonce_str=${nonce}&key=${key}`).toUpperCase()
    ```
    
2. **弹出微信支付**

    需要用到统一下单后返回的 `prePayId` ，重点说一下**第二次签名和第一此签名需要的参数是不一样的**，而且微信官方文档并没有对此进行说明，这里被坑惨了:joy:
    
    ``` javascript
    let nonce = Math.random().toString(36).substr(2)
    let ts = new Date().getTime().toString()
    WeixinJSBridge.invoke(
    'getBrandWCPayRequest', {
      'appId': appId,
      'timeStamp': ts,
      'nonceStr': nonce,
      'package': `prepay_id=${prepayId}`,
      'signType': 'MD5',
      'paySign': md5(`appId=${appId}&nonceStr=${nonce}&package=prepay_id=${prepayId}&signType=MD5&timeStamp=${ts}&key=${key}`).toUpperCase()
    },
    function (res) {
      if (res.err_msg == 'get_brand_wcpay_request:ok') {
        // 处理成功消息
      } else {
        // 处理失败
      }
    })
    ```
    
    > 我踩的坑主要在第二次签名的问题上。如果报 `当前页面的url未注册` 的错误，那么你需要去`商户号 -> 产品中心 -> 开发配置`中添加支付目录

### 总结

到这里，微信公众号网页内支付基本上也就开发完成了，这里面的辛酸苦辣只有自己知道，真的到处都是坑呀。希望大家都可以顺利的完成微信支付开发，不要像我一样踩这么多的坑，愿明天更美好:joy:
