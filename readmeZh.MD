## cloudflare-proxy-anywhere
一个使用Cloudflare解决Umami与Sentry被反广告拦截的Worker

[English](readme.MD) | 中文

## 感谢
基于[vitobotta](https://github.com/vitobotta)的代码进行二次开发，在此表示感谢，原始代码：
https://github.com/umami-software/umami/discussions/1026

## 使用
Key | 必填 | TypeScript类型 | 说明
--- | --- | --- | ---
type |是 | String | 请求类型，GET或者POST
url | 是| String | 被代理JS的URL，不要填写path
urlReplace | 是 | [String,String]  | 访问代理的`path`和访问真实url的`path`，支持正则表达式
paramReplace | 否 | {search:String,replace:String}[] | 对url的参数替换
contextType | 否 | String | 指定返回内容头的 `content-type`
contextReplace | 否 | {search:String,replace:String}[] | 替换返回的`Body`的内容
customAllowHeaders | 否 | string[] | 追加返回的Header里的`Access-Control-Allow-Headers`的内容

## 配置Demo
### Umami配置
首先，让我们假设你的
 - Umami网址是: `https://umami.you.com`  
 - `Cloudflare Worker`网址是: `https://worker.com`
```
    {
        type: "GET",
        url: "https://umami.you.com",
        urlReplace: ["/whatever.js", "/umami.js"],
        contextReplace: [{ search: "/api/collect", replace: "/foo/bar" }],
        contextType: "application/javascript"
    },
    {
        type: "POST",
        url: "https://umami.you.com",
        urlReplace: ["/foo/bar", "/api/collect"]
    }
```
#### Get说明
其中，Get配置为你的网站向你的Cloudflare Worker代理请求的配置

这个配置设置了你访问你的原先Umami的Url将会从  
`https://umami.you.com/umami.js` 变成 `https://worker.com/whatever.js`
这会使大多数将`umami`作为关键词拦截的adblock规则统统失效，当然这是不够的,我们还需要隐藏umami的提交特征  

原先`umami`的上报url是 `https://umami.you.com/api/collect`, 这里使用`contextReplace`将它们统统替换成`/foo/bar`,

最后是返回类型，我们通过`contextType`设置`application/javascript`来让浏览器把它当作Javascript使用

#### Post说明

我们上一步把`/api/collect`通过全局替换变成了`/foo/bar`,

现在Umami脚本会自动向`https://worker.com/foo/bar` Post数据, 我们需要指定转发的Url地址

我们使用`urlReplace`把Url从`/foo/bar`全局替换 `/api/collect`,

现在`Cloudflare Worker` 接收到来自 `https://worker.com/foo/bar` 的Post会自动转发Post到 `https://umami.you.com/api/collect` , 并返回来自umami的返回结果

### Sentry/GlitchTip配置
首先，让我们假设你的
 - Sentry/GlitchTip 后端网址是: `https://sentry.you.com`  
 - Sentry/GlitchTip Js地址是: `https://browser.sentry-cdn.com/7.36.0/bundle.tracing.min.js`
 - `Cloudflare Worker`网址是: `https://worker.com`
```
    {
        type: "GET",
        url: "https://browser.sentry-cdn.com",
        urlReplace: ["/mio.js", "/7.36.0/bundle.tracing.min.js"],
        contextReplace: [{
            search: "${t.host}", replace: "worker.com"
        },
        {
            search: 'sentry_key:t.publicKey,sentry_version:"7",...n&&{sentry_client',
            replace: 'mio_key:t.publicKey,mio_version:"7",...n&&{mio_client'
        }],
        contextType: "application/javascript"
    },
    {
        type: "POST",
        url: "https://sentry.you.com",
        urlReplace: ["/api/1/envelope", "/api/1/envelope/"],
        paramReplace: [
            { search: "mio_key", replace: "sentry_key" },
            { search: "mio_version", replace: "sentry_version" },
            { search: "mio_client", replace: "sentry_client" },
        ],
        contextType: "application/json"
    }
```
 #### Get说明
其中，Get配置为你的网站向你的Cloudflare Worker代理请求的配置

这个配置设置了你访问你的原先 `Sentry/GlitchTip` JS的Url将会从

`https://browser.sentry-cdn.com/7.36.0/bundle.tracing.min.js` 变成 `https://worker.com/mio.js`

这会使大多数将`sentry`作为关键词拦截的Adblock规则统统失效

当然这是不够的,我们还需要隐藏 Sentry/GlitchTip 的Param提交特征

原先 Sentry/GlitchTip 的Param参数为`sentry_key`，`sentry_version`，`sentry_client`，这里使用`contextReplace`将它们统统替换成`mio_key`，`mio_version`，`mio_client`，

最后是返回类型，我们通过`contextType`设置`application/javascript`来让浏览器把它当作Javascript使用

#### Post说明
我们上一步把Param提交参数全部换成`mio_`开头的，现在 Sentry/GlitchTip JS脚本会自动向`https://worker.com/api/1/envelope`Post数据，我们需要指定转发的Url地址。

我们使用`urlReplace`把Url从`/api/1/envelope`全局替换 `/api/1/envelope/`，

这里替换的最后面加了`/`是因为不加的话`Glitch`会进行301跳转

现在`Cloudflare Worker`接收到来自`https://worker.com/api/1/envelope`的Post会自动转发Post到`https://sentry.you.com/api/1/envelope/`，并返回来自 Sentry/GlitchTip 的返回结果

## Buy Me Coffice
- Paypal : https://paypal.me/fudaoyuan
- 支付宝/微信支付 : https://mbd.pub/o/author-bWuWk29q