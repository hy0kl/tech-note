# 删除节点

```javascript
var iframe = document.getElementsByTagName('iframe');
var child  = iframe[0];
child.parentNode.removeChild(child);
```

# 微信开发相关

## 微信浏览器跳转到 app store

*微信在其内置浏览器对 url 做了限制,发现用户要跳转至 app store 下载 app 时,会将其屏蔽*

以下方法止前可以绕过微信的限制.  时间: 2015.01.19  [参考](http://www.ildsea.com/1781.html)

```html
&lt;a href="http://mp.weixin.qq.com/mp/redirect?url=<?php echo urlencode('app store 中特推广下载链接'); ?>"&gt;下载并安装app&lt;/a&gt;
```

## 安卓在微信中下载 apk

将 url 指定为:
`http://app.qq.com/#id=detail&appid=应用id`

