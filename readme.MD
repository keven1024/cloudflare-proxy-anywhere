## cloudflare-proxy-anywhere
A Worker that uses Cloudflare to solve Umami and Sentry by anti-advertising blocking

English | [中文](readmeZh.MD)

## Use
Key | Required | TypeScript type | Description
--- | --- | --- | --- 
type | is | String | the request type, GET or POST
url | Yes | String | The URL of the JS being proxied, do not fill in path
urlReplace | Yes | [String,String] | The `path` to access the proxy and the `path` to access the real url
paramReplace | No | {search:String,replace:String}[] | Parameter replacement for the url
contextType | No | String | Specifies the `content-type` of the returned content header.
contextReplace | no | {search:String,replace:String}[] | Replace the content of the returned `Body`.

## Configuration Demo
### Umami configuration
First, let's assume that your
 - Umami URL is: `https://umami.you.com`  
 - `Cloudflare Worker` URL is: `https://worker.com`
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
#### Get description
Where the Get configuration is the configuration of your site's request to your Cloudflare Worker proxy

This configuration sets the Url that you use to access your original Umami from  
`https://umami.you.com/umami.js` to `https://worker.com/whatever.js`
This will disable most of the adblock rules that block `umami` as a keyword, but of course this is not enough, we also need to hide the umami submission characteristics  

The original `umami` submission URL is `https://umami.you.com/api/collect`, here we use `contextReplace` to replace them all with `/foo/bar`,

Finally, the return type is set to `application/javascript` by `contextType` to make the browser use it as Javascript

#### Post Description

In the previous step we turned `/api/collect` into `/foo/bar` by global replacement,

Now the Umami script will automatically post data to `https://worker.com/foo/bar`, and we need to specify the Url address to forward it to

We use `urlReplace` to replace Url from `/foo/bar` to `/api/collect` globally,

Now `Cloudflare Worker` receive Post from `https://worker.com/foo/bar` will automatically forward Post to `https://umami.you.com/api/collect` , and return the return result from umami

### Sentry/GlitchTip configuration
First, let's assume that your
 - Sentry/GlitchTip backend URL is: ``https://sentry.you.com``  
 - Sentry/GlitchTip Js address is: ``https://browser.sentry-cdn.com/7.36.0/bundle.tracing.min.js``
 - `Cloudflare Worker` URL is: `https://worker.com`
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
 #### Get description
Where Get is configured for your site to request from your Cloudflare Worker proxy

This configuration sets the Url of your original `Sentry/GlitchTip` JS to be accessed from

`https://browser.sentry-cdn.com/7.36.0/bundle.tracing.min.js` to `https://worker.com/mio.js`

This will disable most of the Adblock rules that block `sentry` as a keyword

Of course, this is not enough, we also need to hide the Param submission feature of Sentry/GlitchTip

The original Param parameters of Sentry/GlitchTip are `sentry_key`, `sentry_version`, `sentry_client`, here we use `contextReplace` to replace them all with `mio_key`, `mio_version`, `mio_ client`.

Finally, the return type, we set `application/javascript` by `contextType` to make the browser use it as Javascript

#### Post description
We changed all the Param submission parameters to start with `mio_` in the previous step, and now the Sentry/GlitchTip JS script will automatically Post data to `https://worker.com/api/1/envelope`, and we need to specify the Url address for forwarding.

We use `urlReplace` to globally replace Url from `/api/1/envelope` with `/api/1/envelope/`.

The last part of the replacement here adds `/` because if you don't add it, `Glitch` will do a 301 jump

Now `Cloudflare Worker` will automatically forward the Post from `https://worker.com/api/1/envelope` to `https://sentry.you.com/api/1/envelope/` and return the result from Sentry/ GlitchTip's return result

## Buy Me Coffice
- Paypal : https://paypal.me/fudaoyuan
- Alipay/WePay(Chinese) : https://mbd.pub/o/author-bWuWk29q