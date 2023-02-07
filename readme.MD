## cloudflare-proxy-anywhere
一个使用Cloudflare解决JS被反广告拦截的JS

## 使用
Key | 必填 | TypeScript类型 | 说明
--- | --- | --- | ---
type |是 | String | 请求类型，GET或者POST
url | 是| String | 被代理JS的URL，不要填写path
urlReplace | 是 | [String,String]  | 访问代理的`path`和访问真实url的`path`
paramReplace | 否 | {search:String,replace:String}[] | 对url的参数替换
contextType | 否 | String | 指定返回内容头的 `content-type`
contextReplace | 否 | {search:String,replace:String}[] | 替换返回的`Body`的内容

## 配置Demo
### Umami配置
```
    {
        type: "GET",
        url: "https://umami.fudaoyuan.icu",
        urlReplace: ["/whatever.js", "/umami.js"],
        contextReplace: [{ search: "/api/collect", replace: "/foo/bar" }],
        contextType: "application/javascript"
    },
    {
        type: "POST",
        url: "https://umami.fudaoyuan.icu",
        urlReplace: ["/foo/bar", "/api/collect"]
    },
```
### Sentry配置
```
    {
        type: "GET",
        url: "https://browser.sentry-cdn.com",
        urlReplace: ["/mio.js", "/7.36.0/bundle.tracing.min.js"],
        contextReplace: [{
            search: "${t.host}", replace: "js.fudaoyuan.icu"
        },
        {
            search: 'sentry_key:t.publicKey,sentry_version:"7",...n&&{sentry_client',
            replace: 'mio_key:t.publicKey,mio_version:"7",...n&&{mio_client'
        }],
        contextType: "application/javascript"
    },
    {
        type: "POST",
        url: "https://sentry.fudaoyuan.icu",
        urlReplace: ["/api/1/envelope", "/api/1/envelope"],
        paramReplace: [
            { search: "mio_key", replace: "sentry_key" },
            { search: "mio_version", replace: "sentry_version" },
            { search: "mio_client", replace: "sentry_client" },
        ],
        contextType: "application/json"
    }
```